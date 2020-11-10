# escape=`
ARG SERVERCORE_VERSION=ltsc2019

# Use the latest Windows Server Core image.
FROM mcr.microsoft.com/windows/servercore:${SERVERCORE_VERSION} as msvc

# MSVC Build Tools 2019
RUN powershell.exe -ExecutionPolicy Bypass -Command "iwr https://aka.ms/vs/16/release/vs_buildtools.exe -OutFile vs_buildtools.exe; `
    .\vs_buildtools.exe --quiet --wait --norestart --nocache --add Microsoft.VisualStudio.Workload.VCTools --add Microsoft.VisualStudio.Component.Windows10SDK.18362 --add Microsoft.VisualStudio.Component.VC.CoreBuildTools --add Microsoft.VisualStudio.Component.VC.Redist.14.Latest --add Microsoft.VisualStudio.Component.VC.Tools.x86.x64 --add Microsoft.VisualStudio.Component.VC.CMake.Project | Out-Null; `
    Remove-Item -path .\vs_buildtools.exe"

CMD ["C:\\Program Files (x86)\\Microsoft Visual Studio\\2019\\BuildTools\\VC\\Auxiliary\\Build\\vcvars64.bat", "&&", "cmd"]

FROM msvc as build
ARG QT5_VERSION=5.15.1

# Chocolatey
ENV chocolateyUseWindowsCompression=false
RUN powershell.exe -ExecutionPolicy Bypass -Command "Set-ExecutionPolicy Bypass -Scope Process -Force; [System.Net.ServicePointManager]::SecurityProtocol = [System.Net.ServicePointManager]::SecurityProtocol -bor 3072; iex ((New-Object System.Net.WebClient).DownloadString('https://chocolatey.org/install.ps1'))"

# Applications and setting of path
RUN choco install --yes cmake git activeperl jom `
&& choco install --yes python3 --params '"/InstallDir:C:\Python3"'

RUN powershell.exe -ExecutionPolicy Bypass -Command "[Environment]::SetEnvironmentVariable(\"Path\", [System.Environment]::GetEnvironmentVariable(\"Path\",\"Machine\") + \";C:\Program Files\Git\bin;C:\Python3;C:\Python3\Scripts\", [System.EnvironmentVariableTarget]::Machine )"

# Cloning QT5 sources
WORKDIR C:\
RUN git clone https://code.qt.io/qt/qt5.git qt5-git `
&& cd qt5-git `
&& git checkout %QT5_VERSION% `
&& perl init-repository --module-subset=default,-qtwebengine,-qttools,-qtspeech

SHELL ["cmd","/C"]
# Build Qt5
WORKDIR C:\qtbuild
RUN ""C:\Program Files (x86)\Microsoft Visual Studio\2019\BuildTools\VC\Auxiliary\Build\vcvars64.bat" `
&& C:\qt5-git\configure.bat -release -opensource -confirm-license -static -no-pch -optimize-size -opengl desktop -prefix "C:\Qt5" -skip webengine -nomake tools -nomake tests -nomake examples -skip qtspeech `
&& echo Build on %NUMBER_OF_PROCESSORS% cores `
&& jom /J %NUMBER_OF_PROCESSORS% `
&& jom /J %NUMBER_OF_PROCESSORS% install"

FROM msvc
ARG QT5_VERSION=5.15.1
# Copying Qt5 objects & setting ENV
COPY --from=build C:\Qt5 C:\Qt5\${QT5_VERSION}\msvc2019-static

RUN setx /M Qt5_DIR C:\Qt5\%QT5_VERSION%\msvc2019-static
SHELL ["cmd","/C"]

ENTRYPOINT ""C:\Program Files (x86)\Microsoft Visual Studio\2019\BuildTools\VC\Auxiliary\Build\vcvars64.bat" &&
CMD cmd
