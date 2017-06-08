cmake_minimum_required(VERSION 2.8)

if(NOT DEFINED CMAKE_TOOLCHAIN_FILE)
  if(DEFINED ENV{VITASDK})
    set(CMAKE_TOOLCHAIN_FILE "$ENV{VITASDK}/share/vita.toolchain.cmake" CACHE PATH "toolchain file")
  else()
    message(FATAL_ERROR "Please define VITASDK to point to your SDK path!")
  endif()
endif()


set(SHORT_NAME vitastick)
project(${SHORT_NAME})
include("${VITASDK}/share/vita.cmake" REQUIRED)

set(TITLE_ID "VITASTICK")
set(TITLE_NAME "Vitastick")

set(CMAKE_C_FLAGS "-Wl,-q -Wall -O2 -std=gnu99")
set(CMAKE_CXX_FLAGS "${CMAKE_C_FLAGS} -std=c++11 -fno-rtti -fno-exceptions")
set(VITA_MKSFOEX_FLAGS "${VITA_MKSFOEX_FLAGS} -d PARENTAL_LEVEL=1")


include_directories(
	${PROJECT_SOURCE_DIR}/../skprx/uapi
)

link_directories(
	${CMAKE_CURRENT_BINARY_DIR}
	${PROJECT_SOURCE_DIR}/../skprx/stubs
)


if (NOT ${RELEASE})
	add_definitions(-DENABLE_LOGGING)
endif()


add_executable(${SHORT_NAME}
	main.c
	debugScreenFont.c
)

target_link_libraries(${SHORT_NAME}
	SceDisplay_stub
	SceCtrl_stub
	ScePower_stub
	vitastick_stub
)


vita_create_self(${SHORT_NAME}.self ${SHORT_NAME})
vita_create_vpk(${SHORT_NAME}.vpk ${TITLE_ID} ${SHORT_NAME}.self
  VERSION 01.00
  NAME ${TITLE_NAME}
  FILE sce_sys/icon0.png sce_sys/icon0.png
  FILE sce_sys/livearea/contents/bg0.png sce_sys/livearea/contents/bg0.png
  FILE sce_sys/livearea/contents/startup.png sce_sys/livearea/contents/startup.png
  FILE sce_sys/livearea/contents/template.xml sce_sys/livearea/contents/template.xml
)



add_custom_target(vpksend
	COMMAND curl -T ${SHORT_NAME}.vpk ftp://$(PSVITAIP):1337/ux0:/
	DEPENDS ${SHORT_NAME}.vpk
)

add_custom_target(send
	COMMAND curl -T ${SHORT_NAME} ftp://$(PSVITAIP):1337/ux0:/app/${TITLE_ID}/eboot.bin
	DEPENDS ${SHORT_NAME}
)