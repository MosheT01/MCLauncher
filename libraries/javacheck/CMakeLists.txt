cmake_minimum_required(VERSION 3.1)
project(launcher Java)

# WHY THIS CHANGED: find_package was previously set to Java 1.7.
# Java 21 (the default JDK on macos-latest GitHub Actions runners as of 2024+)
# completely removed support for compiling with -source 7 / -target 7.
# Bumping the minimum required Java to 1.8 makes the build honest about
# what it actually needs, and ensures CMake will error early with a clear
# message if an older JDK is accidentally used instead of a cryptic javac error.
find_package(Java 1.8 REQUIRED COMPONENTS Development)

include(UseJava)
set(CMAKE_JAVA_JAR_ENTRY_POINT JavaCheck)

# WHY THIS CHANGED: The flags -source 7 -target 7 told javac to compile
# class files compatible with Java 7 bytecode. Java 21 removed this option
# entirely (JEP 182 — policy of removing old source/target values after
# several releases). The macOS CI build was therefore failing with:
#   error: Source option 7 is no longer supported. Use 8 or later.
#   error: Target option 7 is no longer supported. Use 8 or later.
# Changing to -source 8 -target 8 is the minimum accepted value on Java 21
# and still produces bytecode that runs on any modern JRE used to launch
# Minecraft (Java 8 class files run on Java 8, 11, 17, 21 without issue).
set(CMAKE_JAVA_COMPILE_FLAGS -target 8 -source 8 -Xlint:deprecation -Xlint:unchecked)

set(SRC
    JavaCheck.java
)

add_jar(JavaCheck ${SRC})
install_jar(JavaCheck "${JARS_DEST_DIR}")
