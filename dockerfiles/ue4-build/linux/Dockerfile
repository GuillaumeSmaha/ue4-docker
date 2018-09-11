ARG TAG

# Since the prebuilt SDL2 library included with UE4.20.0 onwards is not built with ALSA support,
# we rebuild it to include ALSA support (since PulseAudio doesn't play nicely with our containers)
FROM adamrehn/ue4-source:${TAG} AS sdl-builder
RUN apt-get install -y --no-install-recommends libxcursor-dev libxinerama-dev libxi-dev libxrandr-dev x11proto-scrnsaver-dev libwayland-dev libgles2-mesa-dev libpulse-dev libasound2-dev \
    && pip3 install ue4cli

# Perform the build (4.20.0 and newer only, this call is a no-op for older Engine versions)
USER ue4
COPY build-sdl.py /tmp/build-sdl.py
RUN ue4 setroot /home/ue4/UnrealEngine \
    && python3 /tmp/build-sdl.py

# -------

# Isolate the SDL build dependencies in a separate stage to avoid polluting our main image
FROM adamrehn/ue4-source:${TAG}

ENV BUNDLED_LINKER /home/ue4/UnrealEngine/Engine/Extras/ThirdPartyNotUE/SDKs/HostLinux/Linux_x64/v11_clang-5.0.0-centos7/x86_64-unknown-linux-gnu/bin/x86_64-unknown-linux-gnu-ld

# Copy our ALSA-enabled SDL library files (4.20.0 and newer only, the Python call is a no-op for older Engine versions)
ARG SDL_LIBDIR="/home/ue4/UnrealEngine/Engine/Source/ThirdParty/SDL2/SDL-gui-backend/lib/Linux/x86_64-unknown-linux-gnu"
COPY --from=sdl-builder /tmp/sdl /tmp/sdl
COPY copy-sdl.py patch-setup-linux.py linux-root.patch /tmp/

# Install ue4cli and the prerequisites for conan-ue4cli
RUN cd /home/ue4/UnrealEngine \
    && git clone "https://github.com/adamrehn/conan-ue4cli.git" /home/ue4/conan-ue4cli \
    && pip3 install -r /home/ue4/conan-ue4cli/requirements.txt \
    && ue4 setroot /home/ue4/UnrealEngine \
    && python3 /tmp/patch-setup-linux.py /home/ue4/UnrealEngine/Setup.sh \
    && python3 /tmp/patch-setup-linux.py /home/ue4/UnrealEngine/Engine/Build/BatchFiles/Linux/Setup.sh \
    && ./Setup.sh \
    && patch -p 1 < /tmp/linux-root.patch \
    && git config --global user.email "you@example.com" \
    && git commit -a -m "Patch linux root" \
    && (python3 /tmp/copy-sdl.py /tmp/sdl ${SDL_LIBDIR} || true) \
    && ((test -e "$BUNDLED_LINKER" && rm -f "$BUNDLED_LINKER" && ln -fs `which ld` "$BUNDLED_LINKER") || true) \
    && chown -hR ue4 /home/ue4/UnrealEngine \
    && su ue4 -c './Engine/Build/BatchFiles/Linux/Build.sh UE4Editor Linux Development -WaitMutex' \
    && su ue4 -c './Engine/Build/BatchFiles/Linux/Build.sh ShaderCompileWorker Linux Development -WaitMutex' \
    && su ue4 -c './Engine/Build/BatchFiles/Linux/Build.sh UnrealPak Linux Development -WaitMutex' \
    && su ue4 -c 'ue4 setroot /home/ue4/UnrealEngine' \
    && su ue4 -c 'python3 /home/ue4/conan-ue4cli/generate.py'

