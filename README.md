# Contenedores

- Clonar un repositorio que tenga una app Python armada

```bash
git clone https://github.com/rh-aiservices-bu/arc-model.git
```

## Levantar una imagen Python

### Con [Source to image](https://github.com/sclorg/s2i-python-container)

Este concepto es mas 'feliz' y trata de tener una conveción en el codigo, con cada cosa "en su lugar", la construccion se estandariza.

Creamos este Dockerfile (nombre Containerfile.s2i)

```dockerfile
FROM quay.io/fedora/python-310

# Add application sources to a directory that the assemble script expects them
# and set permissions so that the container runs without root access
USER 0
ADD . /tmp/src
RUN /usr/bin/fix-permissions /tmp/src
USER 1001

# Install the dependencies
RUN /usr/libexec/s2i/assemble

# Set the default command for the resulting image
CMD /usr/libexec/s2i/run
```

Como ya tenemos clonada la app podemos armar la imagen haciendo referencia al dir donde esta

```bash
cd containers-python

podman build \
  -t python-test-s2i:latest \
  -f Containerfile.s2i \
  ../arc-model/.
```

Esto crea la imagen que podemos ver haciendo

```bash
podman images | grep python-test-s2i
```

Y luego correr el contenedor

```bash
podman run -d -it \
  -p 8080:8080 \
  --name python-test-s2i \
  python-test-s2i:latest
```

Y testear

```bash
cd containers-python

MY_IMAGE=../arc-model/images/groceries.jpg
MY_ROUTE='localhost:8080'
MY_IMAGE_B64=$(base64 ${MY_IMAGE})
echo "{\"image\": \"${MY_IMAGE_B64}\"}" > jsonfile

curl -X POST "${MY_ROUTE}/predictions" \
  -H "Content-Type: application/json" \
  -d @jsonfile \
  | jq .
```

Para modificar cosas y correr el contenedor de nuevo, tenes que romperlo y volver a crearlo

```bash
podman rm --force python-test-s2i
```

### (Work in Progress) Con Dockerfile mas casero

Por que por ahi necesitas hacer cosas que con source to image te molesta

1. Clonar un repositorio que tenga una app Python armada

```bash
git clone https://github.com/rh-aiservices-bu/arc-model.git
cd arc-model
```

Creas un Dockerfile en el mismo directorio

```dockerfile
FROM quay.io/fedora/python-310

USER 0
COPY --chown=1001:0 requirements.txt /tmp/src
RUN pip install -r requirements.txt

USER 1001

COPY --chown=1001:0 . /tmp/src

CMD []
```
