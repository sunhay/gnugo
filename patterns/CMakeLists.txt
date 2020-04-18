include_directories(${GNUGO_PATH}/patterns)
include_directories(${GNUGO_PATH}/engine)
include_directories(${GNUGO_PATH}/sgf)
include_directories(${GNUGO_PATH}/utils)

# ./mkpat
set(MKPAT_SRC
        "mkpat.c"
        "transform.c"
        "dfa.c")

add_executable(gnugo-mkpat ${MKPAT_SRC})
target_link_libraries(gnugo-mkpat gnugo-utils)

# ./joseki
set(JOSEKI_SRC "joseki.c")
add_executable(gnugo-joseki ${JOSEKI_SRC})
target_link_libraries(gnugo-joseki gnugo-board gnugo-sgf gnugo-utils)

# ./mkeyes
set(MKEYES_SRC "mkeyes.c")
add_executable(gnugo-mkeyes ${MKEYES_SRC})
target_link_libraries(gnugo-mkeyes gnugo-utils)

# ./mkmcpat
set(MKMCPAT_SRC "mkmcpat.c")
add_executable(gnugo-mkmcpat ${MKMCPAT_SRC})
if(UNIX)
    set(PLATFORM_LIBRARIES m)
endif(UNIX)
target_link_libraries(gnugo-mkmcpat gnugo-engine gnugo-sgf gnugo-utils ${PLATFORM_LIBRARIES})

# ./uncompress_fuseki
set(UNCOMPRESS_FUSEKI_SRC "uncompress_fuseki.c")
add_executable(gnugo-uncompress_fuseki ${UNCOMPRESS_FUSEKI_SRC})
target_link_libraries(gnugo-uncompress_fuseki gnugo-utils gnugo-board gnugo-sgf)

# ./extract_fuseki
set(EXTRACT_FUSEKI_SRC "extract_fuseki.c")
add_executable(gnugo-extract_fuseki ${EXTRACT_FUSEKI_SRC})
target_link_libraries(gnugo-extract_fuseki gnugo-engine gnugo-patterns gnugo-engine gnugo-patterns gnugo-sgf gnugo-utils)

# ./compress_fuseki
set(COMPRESS_FUSEKI_SRC "compress_fuseki.c")
add_executable(gnugo-compress_fuseki ${COMPRESS_FUSEKI_SRC})
target_link_libraries(gnugo-compress_fuseki)

# Generate files.

set(JOSEKI_INPUTS "")
set(JOSEKI_NAMES "")
set(GG_BUILT_SOURCES "")
macro(BUILD_JOSEKI NAME PREFIX)
    set(OUTPUT_LOC ${CMAKE_CURRENT_BINARY_DIR}/${NAME})
    set(SOURCE_LOC ${CMAKE_CURRENT_SOURCE_DIR}/${NAME})
    add_custom_command(OUTPUT ${OUTPUT_LOC}.db
                       COMMAND gnugo-joseki ${PREFIX} ${SOURCE_LOC}.sgf > ${OUTPUT_LOC}.db
                       DEPENDS gnugo-joseki ${SOURCE_LOC}.sgf
                       COMMENT "Building joseki: ${OUTPUT_LOC}.db...")
    set(JOSEKI_INPUTS ${JOSEKI_INPUTS} "-i" ${OUTPUT_LOC}.db)
    set(JOSEKI_NAMES ${JOSEKI_NAMES} ${OUTPUT_LOC}.db)
endmacro(BUILD_JOSEKI)

BUILD_JOSEKI(gogo JG)
BUILD_JOSEKI(hoshi_keima JHK)
BUILD_JOSEKI(hoshi_other JHO)
BUILD_JOSEKI(komoku JK)
BUILD_JOSEKI(sansan JS)
BUILD_JOSEKI(mokuhazushi JM)
BUILD_JOSEKI(takamoku JT)

macro(RUN_MKPAT OPTIONS1 OPTIONS2 PATNAME DBNAME CNAME)
    add_custom_command(OUTPUT ${CMAKE_CURRENT_BINARY_DIR}/${CNAME}
                       COMMAND gnugo-mkpat ${OPTIONS1} ${OPTIONS2} ${PATNAME}
                                           -i ${CMAKE_CURRENT_SOURCE_DIR}/${DBNAME}
                                           -o ${CMAKE_CURRENT_BINARY_DIR}/${CNAME}
                       DEPENDS gnugo-mkpat ${CMAKE_CURRENT_SOURCE_DIR}/${DBNAME}
                       COMMENT "Running mkpat: ${dbname} -> ${cname}...")
    set(GG_BUILT_SOURCES ${GG_BUILT_SOURCES} ${CMAKE_CURRENT_BINARY_DIR}/${CNAME})
endmacro(RUN_MKPAT)

macro(RUN_MKPAT_DFA OPTIONS PATNAME DTRNAME DBNAME CNAME)
    add_custom_command(OUTPUT ${CMAKE_CURRENT_BINARY_DIR}/${CNAME}
                       COMMAND gnugo-mkpat -D -m ${OPTIONS}
                                           -t ${CMAKE_CURRENT_SOURCE_DIR}/${DTRNAME} ${PATNAME}
                                           -i ${CMAKE_CURRENT_SOURCE_DIR}/${DBNAME}
                                           -o ${CMAKE_CURRENT_BINARY_DIR}/${CNAME}
                       DEPENDS gnugo-mkpat ${CMAKE_CURRENT_SOURCE_DIR}/${DBNAME}
                                           ${CMAKE_CURRENT_SOURCE_DIR}/${DTRNAME}
                       COMMENT "Running mkpat dfa: ${dbname} -> ${cname}...")
    set(GG_BUILT_SOURCES ${GG_BUILT_SOURCES} ${CMAKE_CURRENT_BINARY_DIR}/${CNAME})
endmacro(RUN_MKPAT_DFA)

macro(RUN_UNCOMPRESS_FUSEKI BOARDSIZE)
    set(FUSEKI_NAME fuseki${BOARDSIZE})
    add_custom_command(OUTPUT ${CMAKE_CURRENT_BINARY_DIR}/${FUSEKI_NAME}.c
                       COMMAND gnugo-uncompress_fuseki ${BOARDSIZE}
                                                       ${CMAKE_CURRENT_SOURCE_DIR}/${FUSEKI_NAME}.dbz
                                                        c > ${CMAKE_CURRENT_BINARY_DIR}/${FUSEKI_NAME}.c
                       DEPENDS gnugo-uncompress_fuseki ${CMAKE_CURRENT_SOURCE_DIR}/${FUSEKI_NAME}.dbz
                       COMMENT "Running uncompress fuseki for size:${BOARDSIZE} -> ${FUSEKI_NAME}...")
    set(GG_BUILT_SOURCES ${GG_BUILT_SOURCES} ${CMAKE_CURRENT_BINARY_DIR}/${FUSEKI_NAME}.c)
endmacro(RUN_UNCOMPRESS_FUSEKI)

## FIXME: It's very ugly that the RUN_MKPAT macro takes two separate
## option arguments, where one is in most cases eliminated by using "".
## The problem with just having one option argument is that specifying
## it as "-c -b" causes the space to be escaped into "-c\ -b". There is
## probably some trivial workaround for someone who actually knows cmake.
RUN_MKPAT(-X "" attpat attack.db apatterns.c)
RUN_MKPAT("" "" defpat defense.db dpatterns.c)
RUN_MKPAT(-b "" handipat handicap.db handipat.c)
RUN_MKPAT(-c "" influencepat influence.db influence.c)
RUN_MKPAT(-c -b barrierspat barriers.db barriers.c)
RUN_MKPAT(-b "" endpat endgame.db endgame.c)
RUN_MKPAT(-c "" conn conn.db conn.c)
RUN_MKPAT(-b "" fusekipat fuseki.db fusekipat.c)
RUN_MKPAT_DFA(-b aa_attackpat aa_attackpats.dtr aa_attackpats.db aa_attackpat.c)
RUN_MKPAT_DFA(-b owl_vital_apat owl_vital_apats.dtr owl_vital_apats.db owl_vital_apat.c)
RUN_MKPAT_DFA(-b owl_attackpat owl_attackpats.dtr owl_attackpats.db owl_attackpat.c)
RUN_MKPAT_DFA(-b owl_defendpat owl_defendpats.dtr owl_defendpats.db owl_defendpat.c)
RUN_UNCOMPRESS_FUSEKI(9)
RUN_UNCOMPRESS_FUSEKI(13)
RUN_UNCOMPRESS_FUSEKI(19)

add_custom_command(OUTPUT ${CMAKE_CURRENT_BINARY_DIR}/josekidb.c
                   COMMAND gnugo-mkpat -C joseki ${JOSEKI_INPUTS} -o ${CMAKE_CURRENT_BINARY_DIR}/josekidb.c
                   DEPENDS gnugo-mkpat ${JOSEKI_NAMES}
                   COMMENT "Running mkpat: josekidb.c...")
set(GG_BUILT_SOURCES ${GG_BUILT_SOURCES} ${CMAKE_CURRENT_BINARY_DIR}/josekidb.c)

add_custom_command(OUTPUT ${CMAKE_CURRENT_BINARY_DIR}/patterns.c
                   COMMAND gnugo-mkpat -b pat
                                       -i ${CMAKE_CURRENT_SOURCE_DIR}/patterns.db
                                       -i ${CMAKE_CURRENT_SOURCE_DIR}/patterns2.db
                                       -o ${CMAKE_CURRENT_BINARY_DIR}/patterns.c
                   DEPENDS gnugo-mkpat ${CMAKE_CURRENT_SOURCE_DIR}/patterns.db ${CMAKE_CURRENT_SOURCE_DIR}/patterns2.db
                   COMMENT "Running mkpat: patterns.db, patterns2.db -> patterns.c...")
set(GG_BUILT_SOURCES ${GG_BUILT_SOURCES} ${CMAKE_CURRENT_BINARY_DIR}/patterns.c)

add_custom_command(OUTPUT ${CMAKE_CURRENT_BINARY_DIR}/eyes.c
                   COMMAND gnugo-mkeyes < ${CMAKE_CURRENT_SOURCE_DIR}/eyes.db
                                        > ${CMAKE_CURRENT_BINARY_DIR}/eyes.c
                   DEPENDS gnugo-mkeyes ${CMAKE_CURRENT_SOURCE_DIR}/eyes.db
                   COMMENT "Running mkeyes: eyes.c...")
set (GG_BUILT_SOURCES ${GG_BUILT_SOURCES} ${CMAKE_CURRENT_BINARY_DIR}/eyes.c)

add_custom_command(OUTPUT ${CMAKE_CURRENT_BINARY_DIR}/mcpat.c
                   COMMAND gnugo-mkmcpat ${CMAKE_CURRENT_SOURCE_DIR}/mc_montegnu_classic.db
                                         ${CMAKE_CURRENT_SOURCE_DIR}/mc_mogo_classic.db
                                         ${CMAKE_CURRENT_SOURCE_DIR}/mc_uniform.db
                                         > ${CMAKE_CURRENT_BINARY_DIR}/mcpat.c
                   DEPENDS gnugo-mkmcpat ${CMAKE_CURRENT_SOURCE_DIR}/mkmcpat.c
                                         ${CMAKE_CURRENT_SOURCE_DIR}/mc_montegnu_classic.db
                                         ${CMAKE_CURRENT_SOURCE_DIR}/mc_mogo_classic.db
                                         ${CMAKE_CURRENT_SOURCE_DIR}/mc_uniform.db
                   COMMENT "Running mkmcpat: mcpat.c...")
set(GG_BUILT_SOURCES ${GG_BUILT_SOURCES} ${CMAKE_CURRENT_BINARY_DIR}/mcpat.c)

# gnugo-patterns
set(PATTERNS_SRC
    "connections.c"
    "helpers.c"
    "transform.c"
    ${GG_BUILT_SOURCES})
add_library(gnugo-patterns STATIC ${PATTERNS_SRC})
