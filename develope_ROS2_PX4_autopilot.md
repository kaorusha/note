# Fast DDS Installation
see [PX4 doc](https://docs.px4.io/master/en/dev_setup/fast-dds-installation.html)
## Prerequisites
### Java JDK 8
Required by `gradle`. Check by running:
```sh
$ java -version
java version "1.8.0_121"
```
[download](https://www.oracle.com/java/technologies/javase/javase-jdk8-downloads.html) `Linux x64 Compressed Archive`
[install](https://docs.oracle.com/javase/8/docs/technotes/guides/install/linux_jdk.html#BJFJJEFG) or install globally by running:
```sh
sudo apt-get install openjdk-8-jdk
```
the above command solved the following error when building `Fast-RTPS-Gen`:
>FAILURE: Build failed with an exception.
>
>* What went wrong:
>Execution failed for task ':idl-parser:compileJava'.
> Could not find tools.jar. Please check that /usr/lib/jvm/java-8-openjdk-amd64 contains a valid JDK installation.


