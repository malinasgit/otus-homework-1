cmake_minimum_required(VERSION 3.12)
#Эта строка устанавливает минимально необходимую версию CMake для выполнения процесса сборки. В данном случае, требуется CMake версии 3.12 или более новой.

set(PATCH_VERSION "1" CACHE INTERNAL "Patch version")
#Здесь устанавливается переменная PATCH_VERSION со значением "1". Опция CACHE INTERNAL означает,
#что переменная будет доступна как кэшированная переменная и будет доступна при повторных вызовах CMake, но не будет видна пользователю.
set(PROJECT_VESRION 0.0.${PATCH_VERSION})
#Создается переменная PROJECT_VERSION, которая зависит от PATCH_VERSION и устанавливает значение "0.0.1" в данном случае.

project(helloworld VERSION ${PROJECT_VESRION})
#Эта строка определяет проект с именем "helloworld" и указывает его версию как значение, рассчитанное на предыдущей строке.

option(WITH_BOOST_TEST "Whether to build Boost test" ON)
#Эта строка создает опцию (переключатель) с именем WITH_BOOST_TEST и описанием "Whether to build Boost test". По умолчанию эта опция установлена в значение ON.

configure_file(version.h.in version.h)
#Эта строка запускает процесс подстановки значений из файла "version.h.in" в файл "version.h".
#Это может использоваться, например, для генерации заголовочных файлов с информацией о версии проекта.

add_executable(helloworld_cli main.cpp)
#Создается исполняемый файл "helloworld_cli" на основе исходного файла "main.cpp".
add_library(helloworld lib.cpp)
#Создается библиотека с именем "helloworld" на основе исходного файла "lib.cpp".

set_target_properties(helloworld_cli helloworld PROPERTIES
    CXX_STANDARD 14
    CXX_STANDARD_REQUIRED ON
)
#Устанавливаются свойства для целей сборки "helloworld_cli" и "helloworld",
#такие как стандарт C++ (CXX_STANDARD) и требования к стандарту (CXX_STANDARD_REQUIRED).

target_include_directories(helloworld
    PRIVATE "${CMAKE_BINARY_DIR}"
)
#Устанавливается директория для включения заголовочных файлов для цели "helloworld".
#В данном случае, это директория "${CMAKE_BINARY_DIR}".

target_link_libraries(helloworld_cli PRIVATE
    helloworld
)
#Устанавливаются зависимости цели "helloworld_cli". Это означает, что "helloworld_cli" будет зависеть от "helloworld".

if(WITH_BOOST_TEST)
    find_package(Boost COMPONENTS unit_test_framework REQUIRED)
    add_executable(test_version test_version.cpp)

    set_target_properties(test_version PROPERTIES
        CXX_STANDARD 14
        CXX_STANDARD_REQUIRED ON
    )

    set_target_properties(test_version PROPERTIES
        COMPILE_DEFINITIONS BOOST_TEST_DYN_LINK
        INCLUDE_DIRECTORIES ${Boost_INCLUDE_DIR}
    )

    target_link_libraries(test_version
        ${Boost_LIBRARIES}
        helloworld
    )
endif()
#Этот блок проверяет значение опции WITH_BOOST_TEST. Если оно установлено ON,
#то выполняется следующий блок кода, который связан с сборкой и тестированием с использованием Boost.

if (MSVC)
    target_compile_options(helloworld_cli PRIVATE
        /W4
    )
    target_compile_options(helloworld PRIVATE
        /W4
    )
    if(WITH_BOOST_TEST)
        target_compile_options(test_version PRIVATE
            /W4
        )
    endif()
else ()
    target_compile_options(helloworld_cli PRIVATE
        -Wall -Wextra -pedantic -Werror
    )
    target_compile_options(helloworld PRIVATE
        -Wall -Wextra -pedantic -Werror
    )
    if(WITH_BOOST_TEST)
        target_compile_options(test_version PRIVATE
            -Wall -Wextra -pedantic -Werror
        )
    endif()
endif()
#Этот блок кода устанавливает опции компиляции в зависимости от того, какая операционная система используется.
#Если это Windows (проверяется по наличию MSVC), то устанавливаются опции компиляции для компилятора Microsoft Visual C++.
#В противном случае (else), устанавливаются опции компиляции для других операционных систем.

install(TARGETS helloworld_cli RUNTIME DESTINATION bin)
#Эта строка указывает, что исполняемый файл "helloworld_cli" должен быть установлен в директорию "bin" после сборки.

set(CPACK_GENERATOR DEB)
set(CPACK_PACKAGE_VERSION_MAJOR "${PROJECT_VERSION_MAJOR}")
set(CPACK_PACKAGE_VERSION_MINOR "${PROJECT_VERSION_MINOR}")
set(CPACK_PACKAGE_VERSION_PATCH "${PROJECT_VERSION_PATCH}")
set(CPACK_PACKAGE_CONTACT baranov131@gmail.com)
include(CPack)
#Эти строки устанавливают генератор пакета CPack для создания пакета Debian (DEB) и включают его в процесс сборки.

if(WITH_BOOST_TEST)
    enable_testing()
    add_test(test_version test_version)
endif()
#Если опция WITH_BOOST_TEST установлена в ON, то этот блок кода включает поддержку
#тестирования с использованием Boost и добавляет тест с именем "test_version".
