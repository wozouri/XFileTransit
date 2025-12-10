find_package(Qt5 REQUIRED COMPONENTS Core Widgets Gui Network)

find_package(Qt5 QUIET COMPONENTS Quick)
if(NOT TARGET Qt5::Quick)
    message(WARNING "Qt5 Quick component not found")
endif()

find_package(Qt5 QUIET COMPONENTS QuickControls2)
if(NOT TARGET Qt5::QuickControls2)
    message(WARNING "Qt5 QuickControls2 component not found")
endif()

set(APP_SOURCES
    main.cpp
)

qt5_add_resources(APP_RES
    ${CMAKE_CURRENT_SOURCE_DIR}/res/res.qrc
)

if(CMAKE_BUILD_TYPE STREQUAL "Debug")
    add_compile_definitions(DEBUG_OUTPUT)
endif()

add_subdirectory(source/driver)
add_subdirectory(source/control)
add_subdirectory(source/model)

add_executable(${PROJECT_NAME}
    ${APP_SOURCES}
    ${APP_RES}
)

# Windows 特有的资源文件
if(WIN32)
    target_sources(${PROJECT_NAME} PRIVATE
        ${CMAKE_CURRENT_LIST_DIR}/res/logo/logo.rc
    )
endif()

if(WIN32 AND MINGW)
    if(CMAKE_BUILD_TYPE STREQUAL "Release")
        set_target_properties(${PROJECT_NAME} PROPERTIES
            LINK_FLAGS "-Wl,-subsystem,windows"
        )
    else()
        set_target_properties(${PROJECT_NAME} PROPERTIES
            LINK_FLAGS "-Wl,-subsystem,console"
        )
    endif()
endif()
target_include_directories(${PROJECT_NAME} PRIVATE
    ${CMAKE_CURRENT_SOURCE_DIR}/include
)


# 链接 Qt 库
target_link_libraries(${PROJECT_NAME} PUBLIC
    Qt5::Core
    Qt5::Gui
    Qt5::Widgets
    Qt5::Network
    Qt5::Quick
    Qt5::QuickControls2
)

target_link_libraries(${PROJECT_NAME} PUBLIC
    driver
    control
    model
)

# 平台特有的系统库
if(WIN32)
    target_link_libraries(${PROJECT_NAME} PUBLIC
        ws2_32
        iphlpapi
    )
else()
    target_link_libraries(${PROJECT_NAME} PUBLIC
        pthread
        dl
    )
endif()

# =========================================================
# 自动复制 OpenSSL DLL 到 exe 目录脚本
# =========================================================
if(WIN32)
    # 1. 再次确认路径 (或者复用之前定义的变量)
    set(OPENSSL_ROOT_DIR "D:/Program Files/OpenSSL-Win64")
    
    # 2. 定义 DLL 文件名 
    # (根据你之前的报错，你是 OpenSSL 3.0 版本)
    set(SSL_DLL "libssl-3-x64.dll")
    set(CRYPTO_DLL "libcrypto-3-x64.dll")

    # 3. 添加构建后命令
    # 【注意】将 "XFileTransit" 替换为你 add_executable 中实际的项目名称
    add_custom_command(TARGET ${PROJECT_NAME} POST_BUILD
        # 复制 libssl
        COMMAND ${CMAKE_COMMAND} -E copy_if_different
            "${OPENSSL_ROOT_DIR}/bin/${SSL_DLL}"
            $<TARGET_FILE_DIR:XFileTransit>
            
        # 复制 libcrypto
        COMMAND ${CMAKE_COMMAND} -E copy_if_different
            "${OPENSSL_ROOT_DIR}/bin/${CRYPTO_DLL}"
            $<TARGET_FILE_DIR:XFileTransit>
            
        COMMENT "正在将 OpenSSL DLL 复制到运行目录..."
    )
endif()