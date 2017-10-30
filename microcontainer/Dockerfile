FROM mono:5.4.0.201 as builder
RUN mkdir /src
RUN echo 'open Suave\n\
[<EntryPoint>]\n\
let main argv =\n\
    let config = { defaultConfig with bindings = [HttpBinding.create HTTP (System.Net.IPAddress.Parse("0.0.0.0")) 8080us] }\n\
    startWebServer config (Successful.OK "Hello World!")\n\
    0\n\
' > /src/hello.fs
RUN echo '<Project Sdk="Microsoft.NET.Sdk">\n\
  <PropertyGroup>\n\
    <OutputType>Exe</OutputType>\n\
    <TargetFramework>net462</TargetFramework>\n\
  </PropertyGroup>\n\
  <ItemGroup>\n\
    <Compile Include="hello.fs" />\n\
  </ItemGroup>\n\
  <ItemGroup>\n\
    <PackageReference Include="suave" Version="2.2.1" />\n\
  </ItemGroup>\n\
</Project>' > /src/hello.fsproj
WORKDIR src
RUN msbuild /t:restore
RUN msbuild /t:build

FROM alpine as bundler
COPY --from=builder /src/bin/Debug/net462/* /
RUN apk add --virtual build-dependencies --repository http://dl-cdn.alpinelinux.org/alpine/edge/testing --no-cache mono mono-dev musl-dev binutils gcc
RUN mkbundle --static --config /etc/mono/config -L /usr/lib/mono/4.5 -o hello hello.exe *.dll /usr/lib/mono/4.5/mscorlib.dll /usr/lib/mono/4.5/System.dll

FROM alpine
COPY --from=bundler /hello /
RUN apk add --no-cache libgcc
EXPOSE 8080
ENTRYPOINT ["/hello"]