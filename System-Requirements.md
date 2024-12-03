# Versity S3 Gateway System Requirements

The Versity S3 Gateway has been built and tested on Linux, Mac, and BSD platforms. While there are no strict processor or memory requirements, the specifications should align with the workload demands. For example, environments with higher simultaneous client access may benefit from additional CPU cores and memory.

## General Validated Hardware Guidelines

**Processor**: x86_64 architecture, 4+ cores recommended for production environments.

**Memory**: Minimum 16 GB RAM (32 GB+ for larger workloads).

**Network**: 10GbE or better network interface is suggested for optimal performance.

## Filesystem Requirements

The only filesystem requirement is support for extended attributes (xattrs).

## Deployment Options

The gateway supports running in Docker for easy containerized deployment, but this is optional and not a strict requirement.
