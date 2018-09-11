ARG TAG
FROM adamrehn/ue4-build:${TAG}

# Create an Installed Build of the Engine and direct ue4cli to use it
RUN cd /home/ue4/UnrealEngine \
    && ./Engine/Build/BatchFiles/RunUAT.sh BuildGraph -target="Make Installed Build Linux" -script=Engine/Build/InstalledEngineBuild.xml -set:HostPlatformOnly=true \
    && ue4 setroot /home/ue4/UnrealEngine/LocalBuilds/Engine/Linux \
    && cp ./Engine/Binaries/Linux/UnrealPak ./LocalBuilds/Engine/Linux/Engine/Binaries/Linux/UnrealPak

