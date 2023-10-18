# CTest的简单使用

- **CTest与GTest的对比**：

  - **CTest**是**CMake**的一个集成**测试工具**，在使用**CMakeLists.txt**构建工程的时候，**CTest**会自动的进行配置，并且给出测试结果。
  - **GTest——Google Test**，是谷歌的一款测试工具，需要单独使用。
  - 考虑便利性，通常会使用**CTest**作为测试工具。

- **如何使用CTest**：

  - **CTest**对**可执行程序**进行测试，因此**CTest**的相关配置代码写到**顶层CMakeLists.txt**中。

  - 启用**CMake**的**CTest**工具：

    - `enable_testing()`命令即可开启**CTest**工具。

  - **添加测试用例**：

    - ```cmake
      add_test(NAME Runs COMMAND Test 25)
      ```

      - 上述命令会添加一个名为`"Runs"`的测试用例，并且这个测试会执行**Test 25**这条指令(不用考虑**Linux**中使用**./**运行)

    - **为测试添加属性**：

      - ```cmake
        set_tests_properties(Runs PROPERTIES PASS_REGULAR_EXPRESSION "25 sqrt is: 5")
        ```

      - 上述命令为名为`"Runs"`的测试用例添加了一条属性`PASS_REGULAR_EXPRESSION`，该属性用来检测**输出信息**中是否**包含特定字符串**(注意这个包含)，这里检测输出信息是否包含**"25 sqrt is 5"**这个字符串。

  - 使用通用的**测试模版**(测试函数)：

    - 观察上述的测试用例，有一些必要条件：

      - **NAME**：测试用例的名称
      - **COMMAND**：测试执行的指令
        - 其中**COMMAND**又分为两部分：
          - **target**：可执行程序名称
          - **arg**：参数
      - **result**：
        - **特定输出字符串**。

    - **构建测试函数**：

      - ```cmake
        function(do_test target arg result)
        	add_test(NAME Runs_${arg} COMMAND ${target} ${arg})
        	set_test_propreties(Runs_${arg} PROPERTIES PASS_REGULAR_EXPRESSION ${result})
        endfunction()
        ```

      - 其中，**do_test**是函数的名称。

    - 测试用例：

      - ```cmake
        do_test(Test 25 "25 sqrt is: 5")
        do_test(Test 9 "9 sqrt is: 3")
        ```

- **顶层CMakeLists.txt**：

  - ```cmake
    cmake_minimum_required(VERSION 3.15)
    
    project(Test VERSION 1.0)
    
    add_library(test_compiler_flags INTERFACE)
    target_compile_features(test_compiler_flags INTERFACE cxx_std_11)
    
    configure_file(TestConfig.h.in TestConfig.h)
    
    add_subdirectory(MathFunctions)
    
    add_executable(Test test.cpp)
    
    target_link_libraries(Test PUBLIC MathFunctions test_compiler_flags)
    
    target_include_directories(Test PUBLIC 
                               "${PROJECT_BINARY_DIR}"
    )
    
    install(TARGETS Test DESTINATION bin)
    install(FILES "${PROJECT_BINARY_DIR}/TestConfig.h" DESTINATION include)
    
    enable_testing()
    #不使用函数添加测试用例：
    add_test(NAME Runs COMMAND Test 25)
    set_tests_properties(Runs PROPERTIES PASS_REGULAR_EXPRESSION "25 sqrt is: 5")
    
    
    function(do_test target arg result)
        add_test(NAME Runs_${arg} COMMAND ${target} ${arg})
        set_tests_properties(Runs_${arg} PROPERTIES PASS_REGULAR_EXPRESSION ${result})
    endfunction()
    
    do_test(Test 25 "25 sqrt is: 5")
    do_test(Test 9 "9 sqrt is: 3")
    ```

- 构建测试过程：

  - ```shell
    cmake --build . #首先构建
    ctest #执行测试
    #相关结果也会在Testing目录中
    ```

  - ```shell
    Test project /home/nickaljy/temp/build
        Start 1: Runs
    1/3 Test #1: Runs .............................   Passed    0.01 sec
        Start 2: Runs_25
    2/3 Test #2: Runs_25 ..........................   Passed    0.01 sec
        Start 3: Runs_9
    3/3 Test #3: Runs_9 ...........................   Passed    0.00 sec
    
    100% tests passed, 0 tests failed out of 3
    
    Total Test time (real) =   0.02 sec
    ```