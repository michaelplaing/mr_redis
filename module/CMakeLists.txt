file(GLOB HEADER_LIST CONFIGURE_DEPENDS "${mister_SOURCE_DIR}/include/mister/*.h")

add_library(
    mr_redis MODULE
    mr_redis.c
    ${HEADER_LIST}
)

target_link_libraries(mr_redis PUBLIC mister)