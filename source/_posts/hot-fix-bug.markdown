---
layout:     post
title:      "浅析Java热修复"
subtitle:   "Java热修复"
date:       2017-06-23
author:     "XuZheng"
# header-img: "post-bg-unix-linux.jpg"
tags:
    - Java
---

> 浅析Java热修复.
### 浅析Java热修复

- 简介

     <p>我们知道大多数的 JVM 具备 Java 的 HotSwap 特性,利用这一特性,我们可以不重启
     Java 进程条件下,改变 Java 方法的实现。通过这种方式,不用停止运行程序,就可以扩展在
     线的应用程序,或者在运行的项目上修复小的错误。</p>
     <p>在项目中我们会有这样的需求，根据科室编号来显示文案信息。但是出现了这种情况，开发人员手滑了或者PMO不明确，把734写成了744。QA人员刚好漏过了这个科室的测试。那么给用户的感觉就是该显示提示的科室不显示了，不该显示的科室反而现实提示。这会很影响用户的体验。</p>

        @HosProcessor(hosCode = "H109472")
        public class TongRenSouthProcessor extends AbstractTonRenProcessor {
            @Override
            protected Set<String> getDeptCodes() {
                // 这里的第一个参数应为734
                return Sets.newHashSet("744", "737", "738", "739", "740", "741", "742", "743", "744",
                        "748", "432", "519", "520", "522", "527");
            }
        }

     <p>修复这样的错误是很简单的。但是如果当时正在业务的高峰期,我们能不能不重启项目，换一种更温柔的方式呢。那么 HotSwap 就给我们提供了另一种解决方案：可以在不重启应用的情况下进行小幅的改动。</p>

- Attach API

     <p>Java程序在运行的时JVM有对其进行管理.因此只要能渗透到对应的JVM  就可以操作上面运行的Java程序。拥有进行这些操作的标准 API就算
     attachment API,它是官方 Java 工具的一部分。使用这个由运行之中的 JVM 所暴露的 API,能让第二个
     Java 进程来同其进行通信。</p>

     <p>其实，我们已经用到了该 API: 它已经由诸如 VisualVM 或者 Java Mission Control 这样的调试
     和模拟工具进行了应用。应用这些附件的 API 并没有同日常使用的标准 Java API 打包在一起，而是被打包到
     了一个特殊的文件之中，叫做 tools.jar，它只包含了一个虚拟机的 JDK 打包发布版本。但是，这个 JAR
     文件的位置并没有进行设置，它在 Windows、Linux，特别是在 Macintosh 上的 VM 都存在差别，不光文件的
     位置，连文件名也各异，有些发行版上就被叫做 classes.jar。最后，IBM 甚至决定对这个 JAR 中包含的一些类
     的名称进行修改，将所有 com.sun 类挪到 com.ibm 命名空间之中, 又添了一个乱子。在 Java 9 中，乱糟糟的
     状态才最终得以清理，tools.jar 被 Jigsaw 的模块 jdk.attach 所替代。</p>

     ![Alt text](http://orzplbg2v.bkt.clouddn.com/img1.png)

     不废话了，就让我们来看看实际的代码是怎样的吧

         static {
                 System.loadLibrary("attach");
         }

         String processId = "13334";//需要修改为要被修复进程的pid
         String jarFileName = getJarFileName();//定位jar文件
         VirtualMachine virtualMachine = VirtualMachine.attach(processId);
         try {
             virtualMachine.loadAgent(jarFileName,"test");
         } catch (AgentLoadException e) {
             e.printStackTrace();
         } catch (AgentInitializationException e) {
             e.printStackTrace();
         }finally {
             virtualMachine.detach();
         }

     通过VirtualMachine.attach(processId)方法我们就可以渗透到对应进程的JVN中了。下来我们通过
     virtualMachine.loadAgent(jarFileName,"test")来加载javaAgent。
     ps（VirtualMachine是在Jre中的lib下的tools.jar中哦）

     在收到一个 JAR 文件之后，目标虚拟机会查看该 JAR 的程序清单描述文件（manifest），并定位处在
     Premain-Class 属性之下的类。这非常类似于 VM 执行一个主方法的方式。有了一个 Java 代理，VM
     和指定的进程 id 就可以查找到一个名为 agentmain 的方法，该方法可以由指定线程中的远程进程来
     执行:

     那么我们来写一段这个代理JAR的代码

        public class HelloWorldAgent {

            public static void agentmain(String arg) {
                System.out.println("Hello, " + arg);
            }

        }

     这个jar文件也需要在manifest文件上加上 Agent-Class:HelloWorldAgent
     使用该API,只要我们知道JVM的进程id，就可以在远程JVM上运行代码，打印出Hello test字符串。甚
     有可能同并不熟 JDK 发行版一部分的 JVM 进行通信，只要附加的 VM 是一个用来访问 tools.jar 的JDK 安装程序。

- Instrumentation API

    利用 Java 代码，即 java.lang.instrument 做动态 Instrumentation 是 Java SE 5 的新特性，它
    把 Java 的 instrument 功能从本地代码中解放出来，使之可以用 Java 代码的方式解决问题。使用 Instrumentation，开发者可以构
    建一个独立于应用程序的代理程序（Agent），用来监测和协助运行在 JVM 上的程序，甚至能够替换和修改某些
    类的定义。有了这样的功能，开发者就可以实现更为灵活的运行时虚拟机监控和Java 类操作了，这样的特性实际
    上提供了一种虚拟机级别支持的 AOP 实现方式，使得开发者无需对 JDK 做任何升级和改动，就可以实现某些
    AOP 的功能了。

    那么在一切都ok的情况下，我们就和VM建立起了通信。但是我们并不是单纯的打印字符串，我们希望可以修复一些
    小的BUG。上面的代码是不能做到的。其实Java代理可以定义第二个参数来接收一个Instrumentation实例。
    Instrumentation中的一个就能够对已经加载的代码进行修改。

   代理的代码便变成了如下的情况：

       public static void agentmain(String arg, Instrumentation inst)
                   throws Exception {
           Class<?> headerUtility = Class.forName("HeaderUtility");
           ByteArrayOutputStream output = new ByteArrayOutputStream();
           try (InputStream input =
                   BugFixAgent.class.getResourceAsStream("/fix/TongRenSouthProcessor.java")) {
               byte[] buffer = new byte[1024];
               int length;
               while ((length = input.read(buffer)) != -1) {
                   output.write(buffer, 0, length);
               }
           }
           instrumentation.redefineClasses(
                   new ClassDefinition(headerUtility, output.toByteArray()));
       }

   为了修复BUG，我们定义了一个TongRenSouthProcessor的修复类，该类的定义了与它替换的类完全相同的字段、方法和修饰符相同。如果
   修复类与要替换的类的定义不同就会导致 UnsupportedOperationException。我们通过instrumentation.redefineClasses
   ( new ClassDefinition(headerUtility, output.toByteArray())),将有BUG的类替换为了我们的修复类。ps(JVM 可能会在应用类重定义时执行 Full GC，并且会对受影响的代码进行重新优化。这会导致程序性能的下降)。

   修复类的代码如下

        @HosProcessor(hosCode = "H109472")
        public class TongRenSouthProcessor extends AbstractTonRenProcessor {

            @Override
            protected Set<String> getDeptCodes() {

                return Sets.newHashSet("734", "737", "738", "739", "740", "741", "742", "743", "744",
                        "748", "432", "519", "520", "522", "527");
            }
        }

   可以看到修复类的定义了与它替换的类的字段、方法和修饰符完全相同,我们只是把744修改为734。
- Byte Buddy

   虽然我们可以通过上述操作修复BUG，但这些操作步骤也并不简单。程序员是很懒的，好在出现了Byte Buddy让我们能很愉快的进行上述操作了Byte Buddy 供了在运行时附加一个代理的便利方法。为了应对 tools.jar 位置和虚拟机类型的差异, Byte Buddy 给附件添加了一个抽象层，在这一层可以自动检测到正确的设置。

   可以通过调用如下代码来附加到对应的进程：

      @Override
      public Instrumentation getInstrumentationByPid(String pid) {
         return ByteBuddyAgent
                 .install(AttachmentProvider.DEFAULT, new ProcessProvider() {
                     @Override
                     public String resolve() {
                         return pid;
                     }
                 });
      }

  让我们追进了install()方法的，核心代码如下

      private static void install(AttachmentProvider attachmentProvider, String processId, AgentProvider agentProvider) {
          AttachmentProvider.Accessor attachmentAccessor = attachmentProvider.attempt();
          if (!attachmentAccessor.isAvailable()) {
              throw new IllegalStateException();
          }
          try {
              Object virtualMachineInstance = attachmentAccessor.getVirtualMachineType()
                      .getDeclaredMethod(ATTACH_METHOD_NAME, String.class)
                      .invoke(STATIC_MEMBER, processId);
              try {
                  attachmentAccessor.getVirtualMachineType()
                          .getDeclaredMethod(LOAD_AGENT_METHOD_NAME, String.class, String.class)
                          .invoke(virtualMachineInstance, agentProvider.resolve().getAbsolutePath(), WITHOUT_ARGUMENTS);
              } finally {
                  attachmentAccessor.getVirtualMachineType().getDeclaredMethod(DETACH_METHOD_NAME).invoke(virtualMachineInstance);
              }
          } catch (RuntimeException exception) {
              throw exception;
          } catch (Exception exception) {
              throw new IllegalStateException("Error during attachment using: " + attachmentProvider, exception);
          }
      }

  attachmentAccessor 对象里面就是刚才上面加入的一些默认的设置，里面主要是设置了一个类型为
  Class<?> clazz = com.sun.tools.attach.VirtualMachine.getClass() 成员,然后通过反射VirtualMachine类里
  面的attach方法，然后传入当前进程号，来attach到当前进程的vm上面，然后通过loadAgent方法，把bytebuddyAgent.jar 加载到进程中
  那么 bytebuddyAgent.jar是从哪里来的呢？ 就是 agentProvider.resolve().getAbsolutePath() 来得到的，他是在代码运行时
  产生的一个临时文件，产生的代码是

      public File resolve() throws IOException {
              File agentJar;
              InputStream inputStream = Installer.class.getResourceAsStream('/' + Installer.class.getName().replace('.', '/') + CLASS_FILE_EXTENSION);
              if (inputStream == null) {
                  throw new IllegalStateException("Cannot locate class file for Byte Buddy installer");
              }
              try {
                  agentJar = File.createTempFile(AGENT_FILE_NAME, JAR_FILE_EXTENSION);
                  agentJar.deleteOnExit(); // Agent jar is required until VM shutdown due to lazy class loading.
                  Manifest manifest = new Manifest();
                  manifest.getMainAttributes().put(Attributes.Name.MANIFEST_VERSION, MANIFEST_VERSION_VALUE);
                  manifest.getMainAttributes().put(new Attributes.Name(AGENT_CLASS_PROPERTY), Installer.class.getName());
                  manifest.getMainAttributes().put(new Attributes.Name(CAN_REDEFINE_CLASSES_PROPERTY), Boolean.TRUE.toString());
                  manifest.getMainAttributes().put(new Attributes.Name(CAN_RETRANSFORM_CLASSES_PROPERTY), Boolean.TRUE.toString());
                  manifest.getMainAttributes().put(new Attributes.Name(CAN_SET_NATIVE_METHOD_PREFIX), Boolean.TRUE.toString());
                  JarOutputStream jarOutputStream = new JarOutputStream(new FileOutputStream(agentJar), manifest);
                  try {
                      jarOutputStream.putNextEntry(new JarEntry(Installer.class.getName().replace('.', '/') + CLASS_FILE_EXTENSION));
                      byte[] buffer = new byte[BUFFER_SIZE];
                      int index;
                      while ((index = inputStream.read(buffer)) != END_OF_FILE) {
                          jarOutputStream.write(buffer, START_INDEX, index);
                      }
                      jarOutputStream.closeEntry();
                  } finally {
                      jarOutputStream.close();
                  }
              } finally {
                  inputStream.close();
              }
              return agentJar;
      }

  这份代码主要就是生成了2个文件，一个是Manifest文件，一个是Installer.class文件。Manifest文件是loadAgent方
  法加载agent.jar的时候，需要去读的一个配置文件，里面有一个属性Agent-Class: net.bytebuddy.agent.Installer 指定了一个类，就是表示要执行这个类的agentmain方法，这个时候loadAgent会传入Instrumentation inst对象，这样在当前的ClassLoader里面就会得到并且保存这个对象了，用于以后class类文件的改。Installer.class文件就是Manifest文件里面指定的class，用于启动agentmain方法的入口类文件，里面就一个Instrumentation inst成员和一个agentmain方法生成了上面的2个文件以后，通过zip方法压缩成一个jar包，保存到临时目录，然后把这个临时目录的绝对地址传入到loadAgent方法，就算完成了agent的启动最后再通过attachmentAccessor.getVirtualMachineType().getDeclaredMethod(DETACH_METHOD_NAME).invoke(virtualMachineInstance);把attach上的jvm进行detach所以通过上面的一系列步骤，就把inst对象加入到了当前的classloader里面，以后再通过这个inst对象，就可以对ClassFileTransformer对象进行操作，完成class文件的修改工作了。

   小编也算是程序员，就让小编来写短段代码吧

   ![](http://orzplbg2v.bkt.clouddn.com/2017-06-23%2015:31:47%E5%B1%8F%E5%B9%95%E6%88%AA%E5%9B%BE.png)

   小编能力差就写了这些接口，虽说沒这并没有什么卵用。但是我就只是装装逼。就不要拆穿我了啦。
