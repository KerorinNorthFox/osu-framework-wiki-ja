A useful shell script to remove a nuget reference to framework and replace with a local checkout (at the same directory depth as your project).

shell:
```shell
CSPROJ="osu.Game/osu.Game.csproj"
SLN="osu.sln"

dotnet remove $CSPROJ package ppy.osu.Framework;
dotnet sln $SLN add ../osu-framework/osu.Framework/osu.Framework.csproj ../osu-framework/osu.Framework.NativeLibs/osu.Framework.NativeLibs.csproj;
dotnet add $CSPROJ reference ../osu-framework/osu.Framework/osu.Framework.csproj
```

powershell:
```powershell
$CSPROJ="osu.Game/osu.Game.csproj"
$SLN="osu.sln"

dotnet remove $CSPROJ package ppy.osu.Framework;
dotnet sln $SLN add ../osu-framework/osu.Framework/osu.Framework.csproj ../osu-framework/osu.Framework.NativeLibs/osu.Framework.NativeLibs.csproj;
dotnet add $CSPROJ reference ../osu-framework/osu.Framework/osu.Framework.csproj
```