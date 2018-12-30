cmake_minimum_required(VERSION 3.5)
project(lua)

aux_source_directory(./src SRC)

add_executable(lua ${SRC})