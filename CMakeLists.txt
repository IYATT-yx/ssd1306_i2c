#[[
    Copyright (C) 2022 IYATT-yx (Zhao Hongfei, 赵洪飞)，2514374431@qq.com

    This program is free software: you can redistribute it and/or modify
    it under the terms of the GNU Affero General Public License as published
    by the Free Software Foundation, either version 3 of the License, or
    (at your option) any later version.

    This program is distributed in the hope that it will be useful,
    but WITHOUT ANY WARRANTY; without even the implied warranty of
    MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
    GNU Affero General Public License for more details.

    You should have received a copy of the GNU Affero General Public License
    along with this program.  If not, see <https://www.gnu.org/licenses/>.
]]

cmake_minimum_required(VERSION 3.16.3)
project(SSD1306_I2C
        VERSION 0.1
        HOMEPAGE_URL github.com/IYATT-yx/ssd1306_i2c)

if (${CMAKE_C_COMPILER_VERSION} VERSION_LESS "8.3.0")
    message(FATAL_ERROR "GNU gcc 版本至少需要 8.3.0")
endif ()

set(CMAKE_C_FLAGS_RELEASE "-std=c17 -O3")
set(CMAKE_C_FLAGS_DEBUG "-std=c17 -Wall -Werror=return-type -Werror=address -Werror=sequence-point -Werror=format-security -Wextra -pedantic -Wimplicit-fallthrough -Wsequence-point -Wswitch-unreachable -Wswitch-enum -Wstringop-truncation -Wbool-compare -Wtautological-compare -Wfloat-equal -Wshadow=global -Wpointer-arith -Wpointer-compare -Wcast-align -Wcast-qual -Wwrite-strings -Wdangling-else -Wlogical-op -Wconversion -g -O0 -DDEBUG")

# 检查是否安装 wiringPi 库,未安装则获取源码安装
# 注： 因为测试环境已经安装 wiringPi，尚未测试此部分是否有效
include(CheckIncludeFiles)
check_include_files(wiringPi.h HAVE_WIRINGPI_HEADS)
if (NOT HAVE_WIRINGPI_HEADS)
    set(WIRINGPI "WiringPi")
    include(FetchContent)
    FetchContent_Declare(${WIRINGPI}
            GIT_REPOSITORY "https://github.com/WiringPi/WiringPi.git"
            SOURCE_DIR /tmp/${WIRINGPI})
    FetchContent_MakeAvailable(${ssd1306})
    # 安装 wiringPi
    execute_process(COMMAND sudo bash /tmp/${WIRINGPI}/INSTALL)
endif ()

# 查找 wiringPi 库
find_path(WIRINGPI_INCLUDE_DIRS NAMES wiringPi.h)
find_library(WIRINGPI_LIBRARIES NAMES wiringPi)

include_directories(${CMAKE_CURRENT_SOURCE_DIR}/include)
aux_source_directory(${CMAKE_CURRENT_SOURCE_DIR}/src SRC)
add_library(ssd1306_i2c SHARED ${SRC})  # 编译库
target_link_libraries(ssd1306_i2c ${WIRINGPI_LIBRARIES}) # 链接 wiringPi

# 安装
set(INSTALL_LIBRARY_PATH "/usr/local/lib")
set(INSTALL_INCLUDE_PATH "/usr/local/include")
# 库文件
install(TARGETS ssd1306_i2c
        LIBRARY DESTINATION ${INSTALL_LIBRARY_PATH})
# 头文件
install(FILES ${CMAKE_CURRENT_SOURCE_DIR}/include/oled_fonts.h
            ${CMAKE_CURRENT_SOURCE_DIR}/include/ssd1306_i2c.h
        DESTINATION ${INSTALL_INCLUDE_PATH})
# pkg-config 信息文件
install(FILES ${CMAKE_CURRENT_SOURCE_DIR}/ssd1306_i2c.pc
        DESTINATION "/usr/local/lib/pkgconfig")
# cmake 信息文件
install(DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}/cmake
        DESTINATION "/usr/local/lib")


# 样例
#######
set(EXECUTABLE_OUTPUT_PATH ${CMAKE_CURRENT_SOURCE_DIR}/bin)  # 编译的可执行文件输出到 bin 目录
# 字符和字符串显示
add_executable(word examples/word.c)
target_link_libraries(word ssd1306_i2c)
# 简单图形
add_executable(figure examples/figure.c)
target_link_libraries(figure ssd1306_i2c)
# 滚动
add_executable(scroll examples/scroll.c)
target_link_libraries(scroll ssd1306_i2c)