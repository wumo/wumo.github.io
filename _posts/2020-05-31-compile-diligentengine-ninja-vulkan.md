---
title:  编译DiligentEngine：Ninja和Vulkan
---

CMake Options:
```
-G Ninja
-DDILIGENT_NO_DIRECT3D11=TRUE
-DDILIGENT_NO_DIRECT3D12=TRUE
-DDILIGENT_NO_OPENGL=TRUE
-DBUILD_SHARED_LIBS=FALSE
```
这里禁用D3D是因为这个[issues](https://github.com/DiligentGraphics/DiligentCore/issues/127)。

应用下面这个patch来消除`ninja $ error`：
```
Index: BuildUtils.cmake
IDEA additional info:
Subsystem: com.intellij.openapi.diff.impl.patch.CharsetEP
<+>UTF-8
===================================================================
--- BuildUtils.cmake	(revision e378bf51d532c15fa56f7688709e0ceacf4c58bd)
+++ BuildUtils.cmake	(date 1590918852310)
@@ -26,11 +26,13 @@
 
         # Copy D3Dcompiler_47.dll 
         if(MSVC)
-            if(WIN64)
-                set(D3D_COMPILER_PATH "\"$(VC_ExecutablePath_x64_x64)\\D3Dcompiler_47.dll\"")
-            else()
-                set(D3D_COMPILER_PATH "\"$(VC_ExecutablePath_x86_x86)\\D3Dcompiler_47.dll\"")
-            endif()
+            get_filename_component(D3D_COMPILER_PATH "${CMAKE_CXX_COMPILER}" DIRECTORY)
+            set(D3D_COMPILER_PATH "${D3D_COMPILER_PATH}\\D3Dcompiler_47.dll")
+#            if(WIN64)
+#                set(D3D_COMPILER_PATH "\"$(VC_ExecutablePath_x64_x64)\\D3Dcompiler_47.dll\"")
+#            else()
+#                set(D3D_COMPILER_PATH "\"$(VC_ExecutablePath_x86_x86)\\D3Dcompiler_47.dll\"")
+#            endif()
             add_custom_command(TARGET ${TARGET_NAME} POST_BUILD
                 COMMAND ${CMAKE_COMMAND} -E copy_if_different
                     ${D3D_COMPILER_PATH}
```
<!-- more -->