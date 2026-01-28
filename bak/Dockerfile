# Use a minimal base image with Go installed
FROM golang:1.21 as builder

# Set the working directory inside the container
WORKDIR /app

# Copy the Go source code
COPY hello.go .

# Build the binary
RUN go build -o hello .

# Use a lightweight final image
FROM gcr.io/distroless/static:nonroot

# Copy the binary from the builder stage
COPY --from=builder /app/hello /

# Set the entrypoint
ENTRYPOINT ["/hello"]
