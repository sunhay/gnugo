set(UTILS_SRC
    "getopt.c"
    "getopt1.c"
    "random.c"
    "gg_utils.c"
    "winsocket.c")

add_library(gnugo-utils STATIC ${UTILS_SRC})
