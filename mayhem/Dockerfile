# Build Stage
FROM ghcr.io/evanrichter/cargo-fuzz:latest as builder

## Add source code to the build stage.
ADD . /src
WORKDIR /src

RUN echo building instrumented harnesses && \
    bash -c "pushd fuzz && cargo +nightly -Z sparse-registry fuzz build && popd" && \
    mv fuzz/target/x86_64-unknown-linux-gnu/release/ss1_1 /ss1_1 && \
    mv fuzz/target/x86_64-unknown-linux-gnu/release/thss_1 /thss_1 && \
    mv fuzz/target/x86_64-unknown-linux-gnu/release/sss_1 /sss_1 && \
    mv fuzz/target/x86_64-unknown-linux-gnu/release/wrapped_secrets_1 /wrapped_secrets_1 && \
    echo done

RUN echo building non-instrumented harnesses && \
    export RUSTFLAGS="--cfg fuzzing -Clink-dead-code -Cdebug-assertions -C codegen-units=1" && \
    bash -c "pushd fuzz && cargo +nightly -Z sparse-registry build --release && popd" && \
    mv fuzz/target/release/ss1_1 /ss1_1_no_inst && \
    mv fuzz/target/release/thss_1 /thss_1_no_inst && \
    mv fuzz/target/release/sss_1 /sss_1_no_inst && \
    mv fuzz/target/release/wrapped_secrets_1 /wrapped_secrets_1_no_inst && \
    echo done

# Package Stage
FROM rustlang/rust:nightly

COPY --from=builder /ss1_1 /ss1_1_no_inst /
COPY --from=builder /thss_1 /thss_1_no_inst /
COPY --from=builder /sss_1 /sss_1_no_inst /
COPY --from=builder /wrapped_secrets_1 /wrapped_secrets_1_no_inst /
