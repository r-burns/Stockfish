add_executable(stockfish
    benchmark.cpp
    bitbase.cpp
    bitboard.cpp
    endgame.cpp
    evaluate.cpp
    main.cpp
    material.cpp
    misc.cpp
    movegen.cpp
    movepick.cpp
    pawns.cpp
    position.cpp
    psqt.cpp
    search.cpp
    thread.cpp
    timeman.cpp
    tt.cpp
    uci.cpp
    ucioption.cpp
    syzygy/tbprobe.cpp
    )

target_link_libraries(stockfish PRIVATE project_all)

add_custom_target(bench
    DEPENDS stockfish
    COMMAND stockfish bench
    )

install(TARGETS stockfish
        RUNTIME DESTINATION ${CMAKE_INSTALL_BINDIR})
