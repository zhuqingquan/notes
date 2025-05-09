[官方文档](https://docs.oracle.com/javase/6/docs/technotes/guides/jni/spec/jniTOC.html)
[语法片段参考](https://gist.github.com/qiao-tw/6e43fb2311ee3c31752e11a4415deeb1)

#### JNI调用java重载的方法
+ JNI中通过GetMethodID方法获取到子类中的方法时，此时直接调用子类的方法。如果用不同类型的子类对象去Call这个方法，则会抛出异常。
+ JNI中通过GetMethodID方法获取到的是基类中的方法时，此时使用任何子类对象调用这个方法，可以产生多态的调用效果

#### JNI FindClass返回null的问题
解决方式：在JVM OnLoad时获取所有用到的class。然后对jclass对象添加NewGlobalRef。这样可以在之后直接使用，而且可以在任意线程中使用。
```
JNI_FIND_CLASS(env, "java/util/Vector", pclass);
gJavaVectorClassJNI.pclass = static_cast<jclass>(env->NewGlobalRef(pclass));
```
+ 极端情况下可能要使用ClassLoader。但是没碰到过。[参考](https://svn.apache.org/repos/asf/mesos/branches/0.10.x/src/java/jni/convert.cpp)

+ 如果是内部类，且内部类不是static，则需要先获取到外部的类才行。[参考](https://blog.csdn.net/lijianhy/article/details/32701343)

#### JNI中的Local reference与Global reference
[资料](https://www.ibm.com/docs/en/sdk-java-technology/8?topic=collector-overview-jni-object-references)
+ 不同的生命周期
==Local references are scoped to their creating stack frame and thread==, and automatically deleted when their creating stack frame returns. ==Global references allow native code to promote a local reference into a form usable by native code in any thread attached to the JVM==.
+ **当在一个线程中创建了一个jobject，然后传递给另一个线程使用时，需要先调用JNIEnv->NewGlobalRef创建全局的引用。否则使用他的那个线程将会抛出异常**
+ 何时需要主动调用DeleteLocalRef：LocalRef是阻止引用被GC，但是当你在本地代码中操作大量对象时，而LocalRefTable又是有限的，及时调用DeleteLocalRef，会释放LocalRef在LocalRefTable中所占位置并使对象及时得到回收。

#### JNI层保存jobject对象
java调用JNI接口时会带上对应的java层对象，JNI层想要保存这个对象的引用以便下次主动调用java类的方法，此时应该先使用**NewGlobalRef**获取jobject对象的引用。如：
```
class HYPlayerEventCallbackAdapter : public hymedia::HYPlayerEventCallback
{
public:
    jobject m_hylivePlayer = nullptr;

    HYPlayerEventCallbackAdapter(JNIEnv* env, jobject pHYLivePlayer)
    {
        m_hylivePlayer = env->NewGlobalRef(pHYLivePlayer);
    }
}

jlong Java_com_huya_sdk_newapi_internal_HYPlayerImp_NativeCreateLivePlayer(JNIEnv* env, jobject pThis, jint playerType)
{
    HYPlayerEventCallbackAdapter* cb = new HYPlayerEventCallbackAdapter(env, pThis);
}
```

#### GetMethodId中方法签名的字符串格式
[参考与例子](https://blog.csdn.net/u010126792/article/details/82348438)

| JAVA类型 | 符号 |
| --- | --- |
| Integer | I |
| boolean | Z |
| Byte | B |
| Char | C |
| Short | S |
| Long | J |
| Float | F |
| Double | D |
| void | V |
| 数组[] | [内部对象 |
| Object对象 | L开头，包名/类名，”;”结尾,$标识嵌套类 |
| String | Ljava/lang/String; |

#### java中持有C++对象指针
将C++中new出的对象指针强制转换为jlong，将jlong返回给java，java需要调用这个对象的方法时传入该值。

#### 获取jclass、jmethodID、jfieldID
```
    //获取Java中的对应的实例类
    jclass jniInfo = (env)->FindClass("com/huya/sdk/api/HYVideoEncConfig");
    if(jniInfo==nullptr)
        return nullptr;
        //获取类构造函数
    jmethodID objectClassInitID = (env)->GetMethodID(jniInfo, "<init>", "()V");
    if(nullptr==objectClassInitID)
        return nullptr;
    jfieldID resolution = (env)->GetFieldID(jniInfo, "mResolution", "I");
    jfieldID fps = (env)->GetFieldID(jniInfo, "mFPS", "I");
    jfieldID codecRate = (env)->GetFieldID(jniInfo, "mCodeRate", "I");
    //jfieldID dConsumeTime = (env)->GetFieldID(jniInfo,"dConsumeTime", "J");
    // 创建对象
    jobject result = (env)->NewObject(jniInfo, objectClassInitID);
    env->DeleteLocalRef(jniInfo)
```

#### 创建java对象
```
    // 创建对象
    jobject result = (env)->NewObject(jniInfo, objectClassInitID);
```

#### 设置java对象Field的值
```
    (env)->SetIntField(result, resolution, cgEncCfg.resolution);
    (env)->SetIntField(result, fps, cgEncCfg.fps);
    (env)->SetIntField(result, codecRate, cgEncCfg.codeRate);
```

#### 创建数组Array并设置数组内容
```
    //获取Java中的对应的实例类
    jclass jniInfo = (env)->FindClass("com/huya/sdk/api/HYVideoEncConfig");
    if(jniInfo==nullptr)
        return nullptr;
    //初始化返回数组
    jobjectArray encCfgs = (env)->NewObjectArray(encCfgVec.size(), jniInfo, NULL);
    for(size_t i=0; i<encCfgVec.size(); i++)
    {
        jobject obj_i = createEncConfigObj(env, encCfgVec[i]);
        //添加到objcet数组中
        (env)->SetObjectArrayElement(encCfgs, i, obj_i);
    }
    LOGT("%s channelsession NativeGetEncConfigs success. size=%lu", kCallLogPrefix, encCfgVec.size());
    env->DeleteLocalRef(jniInfo);
```

#### 代码片段备忘
+ FindClass 获取方法ID fieldID
```c++
// 宏，用于获取pclass下与strMethodName匹配的方法赋值给pMethod
// 注意：如果获取失败将抛出异常。建议在JNI的so加载时调用此方法，有助于在测试时发现so不匹配的问题
#define JNI_GET_JAVA_CLASS_METHOD(env, pclass, pMethod, strMethodName, strMethodSig) \
                                    if(env==nullptr || pclass==nullptr) { pMethod = nullptr; }  \
                                    else {pMethod = env->GetMethodID(pclass, strMethodName , strMethodSig);} \
                                    if(pMethod==nullptr) \
                                    { \
                                        jclass exceptionCls = env->FindClass("java/lang/NoSuchMethodException"); \
                                        env->ThrowNew(exceptionCls, strMethodName); \
                                    } 
                    
#define JNI_FIND_CLASS(env, className, pclass) \
                        jclass pclass = env->FindClass(className); \
                        if (pclass == nullptr) {\
                            jclass exceptionCls = env->FindClass("java/lang/ClassNotFoundException"); \
                            env->ThrowNew(exceptionCls, className); \
                        }

#define JNI_FIND_FIELD(env, pclass, pField, strFieldName, strFieldType) \
                                    if(env==nullptr || pclass==nullptr) { pField = nullptr; }  \
                                    else { pField = env->GetFieldID(pclass, strFieldName, strFieldType); } \
                                    if(nullptr==pField) \
                                    { \
                                        jclass exceptionCls = env->FindClass("java/lang/NoSuchFieldException"); \
                                        env->ThrowNew(exceptionCls, "cdnType"); \
                                    }
```

### 字符串相关
#### 在JNI层获取java string内容的方法
1. JNIEnv.GetStringUTFChars + JNIEnv.ReleaseStringUTFChars
```
const char *key = env->GetStringUTFChars(scryptkey, 0);
env->ReleaseStringUTFChars(scryptkey, key);
```

#### 在JNI层将char*转换成java String对象
```
jstring jMsg = env_->NewStringUTF(msg);
env_->DeleteLocalRef(jMsg);
```