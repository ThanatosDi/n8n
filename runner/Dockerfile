ARG N8N_RUNNERS_VERSION=${N8N_RUNNERS_VERSION}

FROM n8nio/runners:${N8N_RUNNERS_VERSION} AS runner

ARG NODE_INSTALL=""
ARG PYTHON_INSTALL=""
ARG NODE_BUILTIN=""
ARG PYTHON_STDLIB=""

USER root

# 安裝 Node.js 套件（如果有指定）
RUN if [ -n "${NODE_INSTALL}" ]; then \
      cd /opt/runners/task-runner-javascript && pnpm add ${NODE_INSTALL}; \
    fi

# 安裝 Python 套件（如果有指定）
RUN if [ -n "${PYTHON_INSTALL}" ]; then \
      cd /opt/runners/task-runner-python && uv pip install ${PYTHON_INSTALL}; \
    fi

# 產生 n8n-task-runners.json 配置
RUN NODE_ALLOW=$(echo "${NODE_INSTALL}" | tr ' ' ',') && \
    PYTHON_ALLOW=$(echo "${PYTHON_INSTALL}" | tr ' ' ',') && \
    NODE_BUILTIN_ALLOW=$(echo "${NODE_BUILTIN}" | tr ' ' ',') && \
    PYTHON_STDLIB_ALLOW=$(echo "${PYTHON_STDLIB}" | tr ' ' ',') && \
    cat > /etc/n8n-task-runners.json <<EOF
{
	"task-runners": [
		{
			"runner-type": "javascript",
			"workdir": "/home/runner",
			"command": "/usr/local/bin/node",
			"args": [
				"--disallow-code-generation-from-strings",
				"--disable-proto=delete",
				"/opt/runners/task-runner-javascript/dist/start.js"
			],
			"health-check-server-port": "5681",
			"allowed-env": [
				"PATH",
				"GENERIC_TIMEZONE",
				"NODE_OPTIONS",
				"N8N_RUNNERS_AUTO_SHUTDOWN_TIMEOUT",
				"N8N_RUNNERS_TASK_TIMEOUT",
				"N8N_RUNNERS_MAX_CONCURRENCY",
				"N8N_SENTRY_DSN",
				"N8N_VERSION",
				"ENVIRONMENT",
				"DEPLOYMENT_NAME",
				"HOME"
			],
			"env-overrides": {
				"NODE_FUNCTION_ALLOW_BUILTIN": "${NODE_BUILTIN_ALLOW}",
				"NODE_FUNCTION_ALLOW_EXTERNAL": "${NODE_ALLOW}",
				"N8N_RUNNERS_HEALTH_CHECK_SERVER_HOST": "0.0.0.0"
			}
		},
		{
			"runner-type": "python",
			"workdir": "/home/runner",
			"command": "/opt/runners/task-runner-python/.venv/bin/python",
			"args": ["-m", "src.main"],
			"health-check-server-port": "5682",
			"allowed-env": [
				"PATH",
				"N8N_RUNNERS_LAUNCHER_LOG_LEVEL",
				"N8N_RUNNERS_AUTO_SHUTDOWN_TIMEOUT",
				"N8N_RUNNERS_TASK_TIMEOUT",
				"N8N_RUNNERS_MAX_CONCURRENCY",
				"N8N_SENTRY_DSN",
				"N8N_VERSION",
				"ENVIRONMENT",
				"DEPLOYMENT_NAME"
			],
			"env-overrides": {
				"PYTHONPATH": "/opt/runners/task-runner-python",
				"N8N_RUNNERS_STDLIB_ALLOW": "${PYTHON_STDLIB_ALLOW}",
				"N8N_RUNNERS_EXTERNAL_ALLOW": "${PYTHON_ALLOW}"
			}
		}
	]
}
EOF

RUN chown runner:runner /etc/n8n-task-runners.json

USER runner
