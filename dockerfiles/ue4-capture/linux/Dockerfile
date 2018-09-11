ARG TAG
FROM adamrehn/ue4-package:${TAG}


# Patch the problematic UE4 header files under 4.19.x (this call is a no-op under other versions)
RUN git clone "https://github.com/adamrehn/UE4Capture.git" /home/ue4/UE4Capture
    && cd /home/ue4/UE4/Capture/scripts \
    && python3 ./patch-headers.py \
    && cd /home/ue4/UE4Capture/recipes/MediaIPC \
    && python3 ./build.py

