# 🏢 Use organization-approved base image
FROM debian:bullseye-slim

# 🏷️ Comprehensive metadata
LABEL org.opencontainers.image.title="Kong Gateway Golden Image" \
      org.opencontainers.image.version="3.10.0.2-1" \
      org.opencontainers.image.vendor="Your Organization"

# 📥 Copy Kong binary (downloaded separately)
COPY kong.deb /tmp/kong.deb
# COPY kong.deb /tmp/kong.deb

# Install Kong and dependencies
RUN set -ex; \
  apt-get update && \
  apt-get install --yes /tmp/kong.deb && \
  rm -rf /var/lib/apt/lists/* && \
  rm -rf /tmp/kong.deb && \
  \
  # Security hardening
  rm -rf /usr/share/doc/* && \
  rm -rf /usr/share/man/* && \
  \
  # Set proper ownership
  chown kong:0 /usr/local/bin/kong && \
  chown -R kong:0 /usr/local/kong && \
  \
  # Create symlinks
  ln -s /usr/local/openresty/luajit/bin/luajit /usr/local/bin/luajit && \
  ln -s /usr/local/openresty/luajit/bin/luajit /usr/local/bin/lua && \
  ln -s /usr/local/openresty/nginx/sbin/nginx /usr/local/bin/nginx && \
  \
  # Verify installation
  kong version


# Copy custom configurations
COPY configs/kong.conf.template /etc/kong/kong.conf.template
COPY scripts/docker-entrypoint.sh /docker-entrypoint.sh


# Include Custom CA certificate
# COPY myCA.crt /etc/ssl/certs/myCA.crt

# Include custom plugins
# COPY kong-plugin/kong/plugins/ /usr/local/share/lua/5.1/kong/plugins/

# Copy entrypoint script and ensure it is executable
COPY docker-entrypoint.sh /docker-entrypoint.sh
RUN chmod 755 /docker-entrypoint.sh

# Switch to non-root user
USER kong

# Expose only necessary ports (adjust based on your use case)
EXPOSE 8000 8443 8001 8444 8002 8445 8003 8446 8004 8447

# Use a safe stop signal
STOPSIGNAL SIGQUIT

# Healthcheck for Kong
HEALTHCHECK --interval=10s --timeout=10s --retries=3 CMD kong health

# Set the entrypoint
ENTRYPOINT ["/docker-entrypoint.sh"]

# Default command
CMD ["kong", "docker-start"]
