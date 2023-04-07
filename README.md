# rust-crosscompile-docker
Compile your applications for MacOS (aarch64 + x86) and Windows
I was struggeling a lot compiling my applications to Mac and Windows so I researched a bit and collected this as inspiration.

## This is not ment to be used in production but should just be used as an example
PR welcome!

I build a little script that uses the Container like that

```bash
#!/bin/bash

echo "Building magic Docker container!"
docker pull dscso/rust-crosscompiler:latest
echo "Stopping old Dockercontainer"
docker stop crosscompiler
echo "Removing old Dockercontainer"
docker rm crosscompiler
echo "Starting new Dockercontainer, with volumes..."
docker run -v $(pwd)/target-cross:/build/target                               \
           -v $(pwd)/Cargo.toml:/build/Cargo.toml:ro                          \
           -v $(pwd)/src:/build/src:ro                                        \
           --name crosscompiler    -d                                         \
           dscso/rust-crosscompiler:latest                                    \
           sleep infinity

export runincontainer="docker exec -it crosscompiler /bin/bash -c "

echo "Container running... Building application... x86_64-apple-darwin"
$runincontainer "source /entrypoint.sh && cargo build --target=x86_64-apple-darwin --bin client-gui --features gui --config /root/.cargo/config" # --release

echo "Container running... Building application... aarch64-apple-darwin"
$runincontainer "source /entrypoint.sh && cargo build --target=aarch64-apple-darwin --bin client-gui --features gui --config /root/.cargo/config" # --release

echo "Container running... Building application... x86_64-pc-windows-gnu"
$runincontainer "source /entrypoint.sh && cargo build --target=x86_64-pc-windows-gnu --bin client-gui --features gui --config /root/.cargo/config" # --release

docker stop crosscompiler
docker rm crosscompiler
```
