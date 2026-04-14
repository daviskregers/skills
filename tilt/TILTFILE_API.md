# Tiltfile API Reference

## Resource Types

### local_resource

```starlark
local_resource(
    'name',
    cmd='command',              # One-time command
    serve_cmd='server',         # Long-running process
    deps=['file.txt'],          # File deps trigger re-run
    resource_deps=['other'],    # Wait for other resources
    auto_init=True,             # Run on tilt up
    allow_parallel=False,       # Concurrent execution
    readiness_probe=probe(),    # Health check for serve_cmd
    trigger_mode=TRIGGER_MODE_AUTO,
    labels=['group'],           # UI grouping
)
```

`cmd`: runs once, re-runs on changes. `serve_cmd`: long-running, restarted on changes.

### docker_build

```starlark
docker_build(
    'image-name', '.',
    dockerfile='Dockerfile',
    target='stage',
    build_args={'ENV': 'dev'},
    only=['src/', 'go.mod'],
    ignore=['tests/', '*.md'],
    live_update=[...],
)
```

### custom_build

```starlark
custom_build(
    'image-name',
    'bazel build //app:image',
    deps=['src/', 'BUILD'],
    tag='dev',
    skips_local_docker=True,
    live_update=[...],
)
```

### k8s_yaml

```starlark
k8s_yaml('manifests.yaml')
k8s_yaml(['deploy.yaml', 'service.yaml'])
k8s_yaml(helm('chart/', values='values.yaml'))
k8s_yaml(kustomize('overlays/dev'))
k8s_yaml(local('kubectl kustomize .'))
```

### k8s_resource

```starlark
k8s_resource(
    'deployment-name',
    port_forwards='8080:80',
    port_forwards=['8080:80', '9090'],
    resource_deps=['database'],
    objects=['configmap:my-config'],
    labels=['backend'],
    trigger_mode=TRIGGER_MODE_MANUAL,
)
```

### docker_compose / dc_resource

```starlark
docker_compose('docker-compose.yml')
docker_compose(['docker-compose.yml', 'docker-compose.override.yml'])

dc_resource('service-name', resource_deps=['setup'], labels=['services'])
```

## Dependency Ordering

```starlark
# Explicit
k8s_resource('api', resource_deps=['database', 'redis'])
local_resource('migrate', resource_deps=['database'])

# Implicit — image refs create automatic deps
docker_build('myapp', '.')
k8s_yaml('deploy.yaml')  # If uses myapp image, auto-depends
```

### Trigger Modes

```starlark
k8s_resource('expensive-build', trigger_mode=TRIGGER_MODE_MANUAL)
k8s_resource('api', trigger_mode=TRIGGER_MODE_AUTO)  # default
trigger_mode(TRIGGER_MODE_MANUAL)  # set default for all
```

## Live Update

**Step ordering matters:** `fall_back_on()` FIRST → `sync()` → `run()` LAST.

```starlark
docker_build('myapp', '.', live_update=[
    fall_back_on(['package.json', 'package-lock.json']),  # Full rebuild trigger
    sync('./src', '/app/src'),                             # Copy files
    run('npm run build', trigger=['./src']),                # Run after sync
])
```

Steps: `fall_back_on([...])`, `sync(local, container)`, `run(cmd)`, `run(cmd, trigger=[...])`, `run(cmd, echo_off=True)`, `restart_container()`.

## Configuration

```starlark
config.define_string('env', args=True, usage='Environment name')
config.define_bool('debug', usage='Enable debug mode')
config.define_string_list('services', usage='Services to enable')
cfg = config.parse()
env = cfg.get('env', 'dev')
```

Usage: `tilt up -- --env=staging --debug`

### Selective Resources

```starlark
config.set_enabled_resources(['api', 'web'])
```

### Context Validation

```starlark
allow_k8s_contexts(['docker-desktop', 'minikube', 'kind-*'])
ctx = k8s_context()
ns = k8s_namespace()
```

### Default Registry

```starlark
default_registry('gcr.io/my-project')
default_registry('localhost:5000', single_name='dev')
```

## Extensions

```starlark
load('ext://restart_process', 'docker_build_with_restart')
load('ext://namespace', 'namespace_create', 'namespace_inject')
load('ext://git_resource', 'git_checkout')
```

From https://github.com/tilt-dev/tilt-extensions

## Data Handling

```starlark
content = read_file('config.yaml')
data = read_json('config.json')
data = read_yaml('config.yaml')

obj = decode_json('{"key": "value"}')
obj = decode_yaml('key: value')
yaml_list = decode_yaml_stream(multi_doc_yaml)
json_str = encode_json(obj)
yaml_str = encode_yaml(obj)

deployments = filter_yaml(manifests, kind='Deployment')
api = filter_yaml(manifests, name='api')
selected = filter_yaml(manifests, labels={'app': 'myapp'})
```

## File Operations

```starlark
watch_file('config/settings.yaml')
files = listdir('manifests/', recursive=True)

output = local('kubectl get nodes -o name')
local('my-script.sh', env={'DEBUG': '1'})

cwd = os.getcwd()
exists = os.path.exists('file.txt')
joined = os.path.join('dir', 'file.txt')
```

## UI Customization

```starlark
k8s_resource('api', labels=['backend'])
k8s_resource('api', links=[link('http://localhost:8080', 'API')])
```

## Settings

```starlark
update_settings(max_parallel_updates=3, k8s_upsert_timeout_secs=60,
    suppress_unused_image_warnings=['base-image'])

ci_settings(k8s_grace_period='10s', timeout='10m')
```
