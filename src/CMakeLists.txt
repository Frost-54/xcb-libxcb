add_library(libxcb STATIC
      xcb_conn.c 
      xcb_out.c 
      xcb_in.c 
      xcb_ext.c 
      xcb_xid.c
      xcb_list.c 
      xcb_util.c 
      xcb_auth.c
)

find_package(Python3 REQUIRED)

add_custom_target(xcb-gen-headers)
file(GLOB XML_FILES "${CMAKE_CURRENT_SOURCE_DIR}/../../xcb-proto/src/*.xml")

set(INDEX 0)
foreach(FILE IN LISTS XML_FILES)
      get_filename_component(FILE_NAME ${FILE} NAME_WE)
      add_custom_target(xcb-gen-headers-${INDEX}
            DEPENDS ${CMAKE_CURRENT_BINARY_DIR}/${FILE_NAME}.h
      )
      add_custom_command(
            COMMAND cd ${CMAKE_CURRENT_BINARY_DIR}
            COMMAND ${Python3_EXECUTABLE} ${CMAKE_CURRENT_LIST_DIR}/c_client.py -c "1.12" -l "X Version 11" -s "3" -p ${CMAKE_CURRENT_SOURCE_DIR}/../../xcb-proto ${CMAKE_CURRENT_SOURCE_DIR}/../../xcb-proto/src/${FILE_NAME}.xml
            OUTPUT ${CMAKE_CURRENT_BINARY_DIR}/${FILE_NAME}.h
            DEPENDS ${CMAKE_CURRENT_SOURCE_DIR}/../../xcb-proto/src/${FILE_NAME}.xml
      )
      message(STATUS "Generate xlib header ${FILE_NAME}.h from ${FILE_NAME}.xml")
      add_dependencies(xcb-gen-headers xcb-gen-headers-${INDEX})
      math(EXPR INDEX "${INDEX} + 1")
endforeach()
add_dependencies(libxcb x11-root-target)
add_dependencies(x11-root-target xcb-gen-headers)
target_include_directories(x11-root-target INTERFACE ${CMAKE_CURRENT_BINARY_DIR})
