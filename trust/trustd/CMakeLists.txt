project(trustd)

include_directories(
	../../OSX/sec/Security
)

add_lib_ios_shim(libtrustd
	OUTPUT_NAME "trustd"
	MODERN_OBJC
	SOURCES
		CertificateTransparency.m
		md.m
		nameconstraints.c
		OTATrustUtilities.m
		personalization.c
		policytree.c
		SecCAIssuerCache.c
		SecCAIssuerRequest.m
		SecCertificateServer.c
		SecCertificateSource.c
		SecOCSPCache.c
		SecOCSPRequest.c
		SecOCSPResponse.c
		SecPinningDb.m
		SecPolicyServer.c
		SecRevocationDb.c
		SecRevocationNetworking.m
		SecRevocationServer.c
		SecTrustExceptionResetCount.m
		SecTrustLoggingServer.m
		SecTrustServer.c
		SecTrustStoreServer.c
		SecTrustStoreServer.m
		trustd_spi.c
		trustdFileLocations.m
		TrustURLSessionCache.m
		TrustURLSessionDelegate.m

		../../OSX/sec/Security/SecuritydXPC.c
	DEFINITIONS
		LIBTRUSTD=1
)

# This code is written for ARC (default-strong properties, CFBridgingRelease); built without it, the
# URL session delegate's dictionaries were never retained and trustd used them after they were freed.
# ponytail: OTATrustUtilities.m and SecRevocationNetworking.m stay without ARC because ARC makes their
# calls to undeclared methods (MobileAsset stubs, two NSURLSession ones, +URLPathAllowedCharacterSet)
# errors. Declare those in the framework headers, then switch this to OBJC_ARC on the whole library.
set_source_files_properties(
	CertificateTransparency.m
	md.m
	SecCAIssuerRequest.m
	SecPinningDb.m
	SecTrustExceptionResetCount.m
	SecTrustLoggingServer.m
	SecTrustStoreServer.m
	trustdFileLocations.m
	TrustURLSessionCache.m
	TrustURLSessionDelegate.m
	PROPERTIES COMPILE_OPTIONS -fobjc-arc
)

add_darling_executable(trustd
	../../OSX/sec/ipc/server_entitlement_helpers.c
	trustd.c

	# not listed in Xcode project, but necessary
	../../OSX/sec/ipc/util.m
)
target_compile_options(trustd PRIVATE
	-fobjc-arc
)
target_compile_definitions(trustd PRIVATE
	LIBTRUSTD
	SECITEM_SHIM_OSX=1
	SEC_IOS_ON_OSX=1
)
target_link_libraries(trustd
	DER
	bsm.0
	z
	sqlite3
	libtrustd
	utilities
	security_asn1
	login
	CFNetwork
	Network
	Security
	CoreFoundation
	Foundation
	MobileAsset

	# implied dependencies
	objc
	system
)
install(TARGETS trustd DESTINATION libexec/darling/usr/libexec)
install(FILES macOS/com.apple.trustd.plist DESTINATION libexec/darling/System/Library/LaunchDaemons)
install(FILES macOS/com.apple.trustd.agent.plist DESTINATION libexec/darling/System/Library/LaunchAgents)
