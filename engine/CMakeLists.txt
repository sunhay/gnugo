include_directories(${CMAKE_CURRENT_SOURCE_DIR})
include_directories(${GNUGO_PATH}/sgf)
include_directories(${GNUGO_PATH}/patterns)
include_directories(${GNUGO_PATH}/utils)

# gnugo-engine
set(ENGINE_SRC
    "aftermath.c"
    "board.c"
    "boardlib.c"
    "breakin.c"
    "cache.c"
    "clock.c"
    "combination.c"
    "dragon.c"
    "endgame.c"
    "filllib.c"
    "fuseki.c"
    "genmove.c"
    "globals.c"
    "handicap.c"
    "hash.c"
    "influence.c"
    "interface.c"
    "matchpat.c"
    "montecarlo.c"
    "move_reasons.c"
    "movelist.c"
    "optics.c"
    "oracle.c"
    "owl.c"
    "persistent.c"
    "printutils.c"
    "readconnect.c"
    "reading.c"
    "semeai.c"
    "sgfdecide.c"
    "sgffile.c"
    "shapes.c"
    "showbord.c"
    "surround.c"
    "unconditional.c"
    "utils.c"
    "value_moves.c"
    "worm.c")

add_library(gnugo-engine STATIC ${ENGINE_SRC})
target_link_libraries(gnugo-engine gnugo-board gnugo-sgf)

# gnugo-board
SET(BOARD_SRC
    "board.c"
    "boardlib.c"
    "hash.c"
    "printutils.c")

add_library(gnugo-board STATIC ${BOARD_SRC})
target_link_libraries(gnugo-board gnugo-sgf)