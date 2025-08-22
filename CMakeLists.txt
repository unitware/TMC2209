add_library(tmc2209
    src/TMC2209.cpp
)

target_link_libraries(tmc2209
    pico_stdlib
    hardware_uart
    hardware_gpio
    freertos
)

set_target_properties(tmc2209 PROPERTIES COMPILE_FLAGS "-Wall -Wextra -Werror")

target_include_directories(tmc2209 PUBLIC
    src
)
