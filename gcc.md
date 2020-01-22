# ASLR
sudo sysctl kernel.randomize_va_space={0|1|2}

# Position independant executables
-pie -fPIE

# Optimization level
-O{0|1|2}

# Stack canary
-f{no-}stack-protector[-all|-strong]

# source hardening
performs additionnal checks in string and mem functions to avoid overflows and smashing
disable: {-U_FORITY_SOURCE|-D_FORTIFY_SOURCE=0}
enable: -O1 -D_FORTIFY_SOURCE={1|2}

# protection GOT
-Wl,-z,relro,-z,now

-Wl: pass the following options to the linker (comma separated)
relro: RELocation Read-Only
now: tell dynamic linker to resolve all symbols when the program is started and not lazilly (when first called)

=> ld -z relro -z now

# compile warnings
format string atk: -Wformat -Wformat-security

# Full security
-O2 -Wunreachable-code -pedantic -Wextra -Wall -Wformat=2 -D_FORTIFY_SOURCE=2 -fstack-protector-strong -pie -fPIE -Wl,-z,relro,-z,now

--param ssp-buffer-size=4
