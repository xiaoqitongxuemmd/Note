CMake Example

- 根目录

  ```cmake
  # CMake 最低版本号要求
  cmake_minimum_required (VERSION 3.8)
  
  # 项目信息
  project (CppLearningNote)
  
  # 查找当前目录下的所有源文件
  # 并将名称保存到 DIR_SRCS 变量
  aux_source_directory(. DIR_SRCS)
  
  # 添加子目录
  add_subdirectory(TestCode)
  add_subdirectory(Chapter9)
  add_subdirectory(Chapter10)
  add_subdirectory(Chapter12)
  
  # 指定生成目标 
  add_executable(CppLearningNote ${DIR_SRCS})
  
  # 添加链接库
  target_link_libraries(CppLearningNote TestCode)
  target_link_libraries(CppLearningNote Chapter9)
  target_link_libraries(CppLearningNote Chapter10)
  target_link_libraries(CppLearningNote Chapter12)
  ```

- 子目录

  ```cmake
  # 查找当前目录下的所有源文件
  # 并将名称保存到 DIR_LIB_SRCS 变量
  aux_source_directory(. DIR_LIB_SRCS)
  
  # 生成链接库
  add_library (TestCode ${DIR_LIB_SRCS})
  ```

- build 文件

  ```bat
  cmake F:\LearningNote\CppPrimer\Code\ExampleCode -G "Visual Studio 17 2022"
  ```