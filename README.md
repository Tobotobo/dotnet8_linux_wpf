# dotnet8_linux_wpf

## 概要

.NET 7 で WPF を Linux ビルドできるようになっているという情報を入手したため  
本当にできるのか実験した。  
なお、本実験は .NET 8 rc で実施している。  

## 結果概要
できる

## 環境
```
# cat /etc/os-release
PRETTY_NAME="Ubuntu 22.04.3 LTS"
NAME="Ubuntu"
VERSION_ID="22.04"
VERSION="22.04.3 LTS (Jammy Jellyfish)"
VERSION_CODENAME=jammy
ID=ubuntu
ID_LIKE=debian
HOME_URL="https://www.ubuntu.com/"
SUPPORT_URL="https://help.ubuntu.com/"
BUG_REPORT_URL="https://bugs.launchpad.net/ubuntu/"
PRIVACY_POLICY_URL="https://www.ubuntu.com/legal/terms-and-policies/privacy-policy"
UBUNTU_CODENAME=jammy
```
```
# dotnet --version
8.0.100-rc.1.23455.8
```

## メモ

### Linux にインストールした dotnet には wpf のテンプレートが無いため、Windows から引っこ抜いてインストールした。
C:\Program Files\dotnet\templates\8.0.0-rc.1.23421.29\microsoft.dotnet.wpf.projecttemplates.8.0.0-rc.1.23420.4.nupkg
```
dotnet new --install ./microsoft.dotnet.wpf.projecttemplates.8.0.0-rc.1.23420.4.nupkg 
```

### dotnet8_linux_wpf.csproj の PropertyGroup に以下を追加
```
<EnableWindowsTargeting>true</EnableWindowsTargeting>
```

### ビルド
```
dotnet build
```
普通にビルドは通るが生成されるのは dll なので次項のパブリッシュへ
```
# dotnet build
MSBuild のバージョン 17.8.0-preview-23418-03+0125fc9fb (.NET)
  復元対象のプロジェクトを決定しています...
  /home/dotnet8_linux_wpf/dotnet8_linux_wpf.csproj を復元しました (984 ms)。
/usr/share/dotnet/sdk/8.0.100-rc.1.23455.8/Sdks/Microsoft.NET.Sdk/targets/Microsoft.NET.RuntimeIdentifierInference.targets(311,5): message NETSDK1057: プレビュー版の .NET を使用しています。https://aka.ms/dotnet-support-policy をご覧ください [/home/dotnet8_linux_wpf/dotnet8_linux_wpf.csproj]
  dotnet8_linux_wpf -> /home/dotnet8_linux_wpf/bin/Debug/net8.0-windows/dotnet8_linux_wpf.dll

ビルドに成功しました。
    0 個の警告
    0 エラー

経過時間 00:00:06.22
```

### パブリッシュ
```
dotnet publish -r win-x64 --no-self-contained
```
```
# dotnet publish -r win-x64 --no-self-contained
MSBuild のバージョン 17.8.0-preview-23418-03+0125fc9fb (.NET)
  復元対象のプロジェクトを決定しています...
  /home/dotnet8_linux_wpf/dotnet8_linux_wpf.csproj を復元しました (304 ms)。
/usr/share/dotnet/sdk/8.0.100-rc.1.23455.8/Sdks/Microsoft.NET.Sdk/targets/Microsoft.NET.RuntimeIdentifierInference.targets(311,5): message NETSDK1057: プレビュー版の .NET を使用しています。https://aka.ms/dotnet-support-policy をご覧ください [/home/dotnet8_linux_wpf/dotnet8_linux_wpf.csproj]
  dotnet8_linux_wpf -> /home/dotnet8_linux_wpf/bin/Release/net8.0-windows/win-x64/dotnet8_linux_wpf.dll
  dotnet8_linux_wpf -> /home/dotnet8_linux_wpf/bin/Release/net8.0-windows/win-x64/publish/
```
exe が生成されている
```
bin/Release/net8.0-windows/win-x64# tree
.
├── dotnet8_linux_wpf.deps.json
├── dotnet8_linux_wpf.dll
├── dotnet8_linux_wpf.exe
├── dotnet8_linux_wpf.pdb
├── dotnet8_linux_wpf.runtimeconfig.json
└── publish
    ├── dotnet8_linux_wpf.deps.json
    ├── dotnet8_linux_wpf.dll
    ├── dotnet8_linux_wpf.exe
    ├── dotnet8_linux_wpf.pdb
    └── dotnet8_linux_wpf.runtimeconfig.json
```
Windows に落として実行したところ実行できた
![](doc/image/スクリーンショット%202023-09-27%20102602.png)

### 自己完結型にできるか？
→　できる
```
dotnet publish -r win-x64 --self-contained
```
```
# dotnet publish -r win-x64 --self-contained
MSBuild のバージョン 17.8.0-preview-23418-03+0125fc9fb (.NET)
  復元対象のプロジェクトを決定しています...
  /home/dotnet8_linux_wpf/dotnet8_linux_wpf.csproj を復元しました (299 ms)。
/usr/share/dotnet/sdk/8.0.100-rc.1.23455.8/Sdks/Microsoft.NET.Sdk/targets/Microsoft.NET.RuntimeIdentifierInference.targets(311,5): message NETSDK1057: プレビュー版の .NET を使用しています。https://aka.ms/dotnet-support-policy をご覧ください [/home/dotnet8_linux_wpf/dotnet8_linux_wpf.csproj]
  dotnet8_linux_wpf -> /home/dotnet8_linux_wpf/bin/Release/net8.0-windows/win-x64/dotnet8_linux_wpf.dll
  dotnet8_linux_wpf -> /home/dotnet8_linux_wpf/bin/Release/net8.0-windows/win-x64/publish/
```
### 自己完結 + シングルバイナリ
→　できるが相変わらず周囲に dll が残っていてシングルじゃない
```
dotnet publish -r win-x64 -p:PublishSingleFile=true --self-contained
```
```
# dotnet publish -r win-x64 -p:PublishSingleFile=true --self-contained
MSBuild のバージョン 17.8.0-preview-23418-03+0125fc9fb (.NET)
  復元対象のプロジェクトを決定しています...
  /home/dotnet8_linux_wpf/dotnet8_linux_wpf.csproj を復元しました (874 ms)。
/usr/share/dotnet/sdk/8.0.100-rc.1.23455.8/Sdks/Microsoft.NET.Sdk/targets/Microsoft.NET.RuntimeIdentifierInference.targets(311,5): message NETSDK1057: プレビュー版の .NET を使用しています。https://aka.ms/dotnet-support-policy をご覧ください [/home/dotnet8_linux_wpf/dotnet8_linux_wpf.csproj]
  dotnet8_linux_wpf -> /home/dotnet8_linux_wpf/bin/Release/net8.0-windows/win-x64/dotnet8_linux_wpf.dll
  dotnet8_linux_wpf -> /home/dotnet8_linux_wpf/bin/Release/net8.0-windows/win-x64/publish/
```
```
bin/Release/net8.0-windows/win-x64/publish# du -k * | awk '{printf "%-30s %10s KB\n", $2, $1}'
D3DCompiler_47_cor3.dll              4804 KB
PenImc_cor3.dll                       156 KB
PresentationNative_cor3.dll          1208 KB
dotnet8_linux_wpf.exe              149816 KB
dotnet8_linux_wpf.pdb                  16 KB
vcruntime140_cor3.dll                 108 KB
wpfgfx_cor3.dll                      1916 KB
```
### 自己完結 + シングルバイナリ + トリミング
→　ダメ。WPFはトリミング不可
https://learn.microsoft.com/ja-jp/dotnet/core/deploying/trimming/trim-self-contained
```
dotnet publish -r win-x64 -p:PublishSingleFile=true -p:PublishTrimmed=true --self-contained
```
```
# dotnet publish -r win-x64 -p:PublishSingleFile=true -p:PublishTrimmed=true --self-contained
MSBuild のバージョン 17.8.0-preview-23418-03+0125fc9fb (.NET)
  復元対象のプロジェクトを決定しています...
  復元対象のすべてのプロジェクトは最新です。
/usr/share/dotnet/sdk/8.0.100-rc.1.23455.8/Sdks/Microsoft.NET.Sdk/targets/Microsoft.NET.RuntimeIdentifierInference.targets(257,5): error NETSDK1168: WPF is not supported or recommended with trimming enabled. Please go to https://aka.ms/dotnet-illink/wpf for more details. [/home/dotnet8_linux_wpf/dotnet8_linux_wpf.csproj]
```
### NativeAOT
→　やっぱ無理  
https://learn.microsoft.com/en-us/dotnet/core/deploying/trimming/incompatibilities
```
dotnet publish -r win-x64 -p:PublishAot=true
```
```
# dotnet publish -r win-x64 -p:PublishAot=true
MSBuild のバージョン 17.8.0-preview-23418-03+0125fc9fb (.NET)
  復元対象のプロジェクトを決定しています...
  /home/dotnet8_linux_wpf/dotnet8_linux_wpf.csproj を復元しました (23.39 sec)。
/usr/share/dotnet/sdk/8.0.100-rc.1.23455.8/Sdks/Microsoft.NET.Sdk/targets/Microsoft.NET.RuntimeIdentifierInference.targets(257,5): error NETSDK1168: WPF is not supported or recommended with trimming enabled. Please go to https://aka.ms/dotnet-illink/wpf for more details. [/home/dotnet8_linux_wpf/dotnet8_linux_wpf.csproj]
```


## 参考
- .NET 7 で WPF を Linux ビルドする  
  https://tech.guitarrapc.com/entry/2022/11/11/031555