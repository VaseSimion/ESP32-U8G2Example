idf_component_register(SRCS "main.cpp"
                    INCLUDE_DIRS "."
                    REQUIRES u8g2 esp_driver_i2c esp_driver_gpio)