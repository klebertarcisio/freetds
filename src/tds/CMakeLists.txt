add_subdirectory(unittests)

if(PERL_FOUND AND NOT EXISTS "${CMAKE_SOURCE_DIR}/configure")
	# get name of null device
	set(NULLNAME "/dev/null")
	if(CMAKE_SYSTEM_NAME STREQUAL "Windows")
		set(NULLNAME "NUL:")
	endif(CMAKE_SYSTEM_NAME STREQUAL "Windows")

	set(ENC_SOURCE_DIR "${CMAKE_SOURCE_DIR}/include/freetds")

	add_custom_command(OUTPUT tds_willconvert.h
		COMMAND ${PERL_EXECUTABLE} tds_willconvert.pl ../../include/freetds/proto.h > tds_willconvert.h
		MAIN_DEPENDENCY tds_willconvert.pl
		WORKING_DIRECTORY ${CMAKE_CURRENT_LIST_DIR})
	add_custom_command(OUTPUT "${CMAKE_BINARY_DIR}/include/freetds/encodings.h"
		COMMAND ${PERL_EXECUTABLE} "${ENC_SOURCE_DIR}/encodings.pl" "${ENC_SOURCE_DIR}" ${GPERF} > encodings.h 2> ${NULLNAME}
		MAIN_DEPENDENCY "${ENC_SOURCE_DIR}/encodings.pl"
		DEPENDS "${ENC_SOURCE_DIR}/alternative_character_sets.h" "${ENC_SOURCE_DIR}/character_sets.h"
		WORKING_DIRECTORY "${CMAKE_BINARY_DIR}/include/freetds")
	add_custom_command(OUTPUT num_limits.h
		COMMAND ${PERL_EXECUTABLE} num_limits.pl > num_limits.h
		MAIN_DEPENDENCY num_limits.pl
		WORKING_DIRECTORY ${CMAKE_CURRENT_LIST_DIR})
	add_custom_command(OUTPUT tds_types.h
		COMMAND ${PERL_EXECUTABLE} types.pl ../../misc/types.csv ../../include/freetds/proto.h > tds_types.h
		MAIN_DEPENDENCY types.pl
		DEPENDS ../../misc/types.csv ../../include/freetds/proto.h
		WORKING_DIRECTORY ${CMAKE_CURRENT_LIST_DIR})
endif(PERL_FOUND AND NOT EXISTS "${CMAKE_SOURCE_DIR}/configure")

if(WIN32)
	set(add_SRCS sspi.c)
else(WIN32)
	set(add_SRCS challenge.c gssapi.c)
endif(WIN32)

add_library(tds STATIC
	mem.c token.c util.c login.c read.c
        write.c convert.c numeric.c config.c query.c iconv.c
        locale.c vstrbuild.c
        getmac.c data.c net.c tls.c
        tds_checks.c log.c
        bulk.c packet.c stream.c random.c
        sec_negotiate_gnutls.h sec_negotiate_openssl.h sec_negotiate.c
	tds_willconvert.h num_limits.h tds_types.h
	"${CMAKE_BINARY_DIR}/include/freetds/encodings.h"
	${add_SRCS}
)
if (NOT WIN32)
	set_target_properties(tds PROPERTIES POSITION_INDEPENDENT_CODE ON)
endif()

if(WIN32)
INSTALL(TARGETS tds
	PUBLIC_HEADER DESTINATION include
		RUNTIME DESTINATION bin
		LIBRARY DESTINATION lib
		ARCHIVE DESTINATION lib
	)
endif(WIN32)
