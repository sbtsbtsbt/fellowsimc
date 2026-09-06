##################
# 'engine' library
##################

# define source files
include(../source_files/cmake_engine.txt)


# For w/e reason it includes files that don't exist like `class_modules/apl/apl_death_knight.hpp` (cmake_engine.txt?).
# Just add all files here
set(files)
foreach(it IN LISTS source_files)
  if(EXISTS "${CMAKE_CURRENT_SOURCE_DIR}/${it}")
    list(APPEND files "${it}")
  endif()
endforeach()


# Now include the ship files
list(APPEND files
  class_modules/fs_player.cpp
  class_modules/sc_aeona.cpp
  class_modules/sc_ardeos.cpp
  class_modules/sc_elarion.cpp
  class_modules/sc_gunde.cpp
  class_modules/sc_mara.cpp
  class_modules/sc_rime.cpp
  class_modules/sc_tariq.cpp
  class_modules/sc_xavian.cpp
)

set(source_files ${files})

# Define engine library
add_library(engine ${source_files})
target_include_directories(engine PUBLIC . ./include ./lib)
sc_common_compiler_options(engine)

# Cross compile insanity (@todo put in header?).
if(NOT MSVC)
  target_compile_definitions(engine PRIVATE __popcnt64=__builtin_popcountll)
endif()

# Make cmake selections visible to C++ code
if(SC_NO_THREADING)
    target_compile_definitions(engine PUBLIC SC_NO_THREADING)
endif()
if(SC_NO_NETWORKING)
    target_compile_definitions(engine PUBLIC SC_NO_NETWORKING)
endif()

# Detect pthreads
if(NOT SC_NO_THREADING)
    set(THREADS_PREFER_PTHREAD_FLAG ON)
    find_package(Threads REQUIRED)
    target_link_libraries(engine Threads::Threads)
endif()

# Networking libs
if(NOT SC_NO_NETWORKING)
    if(NOT WIN32)
        find_package(CURL REQUIRED)
        if (CURL_FOUND)
            target_link_libraries(engine ${CURL_LIBRARIES})
            target_include_directories(engine PUBLIC ${CURL_INCLUDE_DIRS})
        else(CURL_FOUND)
            message(FATAL_ERROR "libcurl not found")
        endif()
    else()
        target_link_libraries(engine crypt32)
        target_link_libraries(engine wininet)
    endif()
endif()

if(SC_EVENT_QUEUE_DEBUG)
    target_compile_definitions(engine PUBLIC EVENT_QUEUE_DEBUG)
endif()

target_compile_definitions(engine PRIVATE "SC_GIT_REV=\"${GIT_COMMIT_HASH}\"" "SC_GIT_BRANCH=\"${GIT_BRANCH}\"")

add_custom_target(file_toucher
        COMMAND ${CMAKE_COMMAND} -E touch_nocreate ${CMAKE_CURRENT_SOURCE_DIR}/util/git_info.cpp)

add_dependencies(engine file_toucher)

# Default Blizzard Community Platform API key
if (DEFINED SC_DEFAULT_APIKEY)
  target_compile_definitions(engine PRIVATE "SC_DEFAULT_APIKEY=\"${SC_DEFAULT_APIKEY}\"")
endif()
