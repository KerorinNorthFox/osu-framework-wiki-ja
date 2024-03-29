A useful shell script to remove a nuget reference to framework and replace with a local checkout (at the same directory depth as your project).

shell:
```shell
CSPROJ="osu.Game/osu.Game.csproj"
SLN="osu.sln"

dotnet remove $CSPROJ package ppy.osu.Framework
dotnet sln $SLN add ../osu-framework/osu.Framework/osu.Framework.csproj ../osu-framework/osu.Framework.NativeLibs/osu.Framework.NativeLibs.csproj
dotnet add $CSPROJ reference ../osu-framework/osu.Framework/osu.Framework.csproj

SLNF="osu.Desktop.slnf"
tmp=$(mktemp)
jq '.solution.projects += ["../osu-framework/osu.Framework/osu.Framework.csproj", "../osu-framework/osu.Framework.NativeLibs/osu.Framework.NativeLibs.csproj"]' osu.Desktop.slnf > $tmp
mv -f $tmp $SLNF
```

powershell:
```powershell
$CSPROJ="osu.Game/osu.Game.csproj"
$SLN="osu.sln"

dotnet remove $CSPROJ package ppy.osu.Framework;
dotnet sln $SLN add ../osu-framework/osu.Framework/osu.Framework.csproj ../osu-framework/osu.Framework.NativeLibs/osu.Framework.NativeLibs.csproj;
dotnet add $CSPROJ reference ../osu-framework/osu.Framework/osu.Framework.csproj

$SLNF=Get-Content "osu.Desktop.slnf" | ConvertFrom-Json
$TMP=New-TemporaryFile
$SLNF.solution.projects += ("../osu-framework/osu.Framework/osu.Framework.csproj", "../osu-framework/osu.Framework.NativeLibs/osu.Framework.NativeLibs.csproj")
ConvertTo-Json $SLNF | Out-File $TMP -Encoding UTF8
Move-Item -Path $TMP -Destination "osu.Desktop.slnf" -Force
```

in **addition**, for iOS:

shell:
```shell
PROPS="osu.iOS.props";
CSPROJ="osu.iOS/osu.iOS.csproj";
SLN="osu.sln";
sed -i "" -e "s/<PackageReference Include=\"ppy\.osu\.Framework\".*$/<ProjectReference Include=\"..\/..\/osu-framework\/osu.Framework\/osu.Framework.csproj\" \/>/g" $PROPS;
sed -i "" -e "s/<PackageReference Include=\"ppy\.osu\.Framework\.iOS\".*$/<ProjectReference Include=\"..\/..\/osu-framework\/osu.Framework.iOS\/osu.Framework.iOS.csproj\" \/>/g" $PROPS;
dotnet sln $SLN add ../osu-framework/osu.Framework/osu.Framework.csproj ../osu-framework/osu.Framework.iOS/osu.Framework.iOS.csproj ../osu-framework/osu.Framework.NativeLibs/osu.Framework.NativeLibs.csproj;
```

in **addition**, for Android:

powershell:

```powershell
$PROPS="osu.Android.props"
$SLN="osu.sln"

dotnet sln $SLN add ../osu-framework/osu.Framework.Android/osu.Framework.Android.csproj;
((Get-Content -Path $PROPS) -replace '<PackageReference Include="ppy\.osu\.Framework\.Android" .*',"<ProjectReference Include=`"../../osu-framework/osu.Framework/osu.Framework.csproj`" />`n    <ProjectReference Include=`"../../osu-framework/osu.Framework.Android/osu.Framework.Android.csproj`" />") | Set-Content -Path $PROPS
```