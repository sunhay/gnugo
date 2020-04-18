include_directories(${GNUGO_PATH}/utils)

set(SGF_SRC
    "sgf_utils.c"
    "sgfnode.c"
    "sgftree.c")

add_library(gnugo-sgf STATIC ${SGF_SRC})
