# Cluster NodeJS Express

<p align="left">
<img src="https://img.shields.io/badge/STATUS-EN DESARROLLO-orange">
</p>

## Índice

1. [Dependencias](#dependencias)
2. [Uso de Cluster](#uso-de-cluster)
3. [Metricas de rendimiento](#metricas-de-rendimiento)
4. [Uso de PM2 para administrar un clúster de Node.js](#uso-de-pm2-para-administrar-un-clúster-de-node.js)
5. [Cuestiones](#cuestiones)

---

## 1. Dependencias

Lo primero de todo necesitaremos actualizar el sistema e instalar curl ya que como Bullseye trae una versión vieja por defecto.
Entonces haremos 

```
sudo apt update
sudo apt install -y curl
```

Ahora instalaremos Node.js y npm. En este caso instalaremos la versión 22.x de Nodejs.

```
curl -fsSL https://deb.nodesource.com/setup_22.x | sudo -E bash -
````

Y posterior a esto instalaremos nodejs:

```
sudo apt-get install -y nodejs
```

Y tambien npm:

```
sudo apt install npm
```

![imagenver](../doc/img/cp1.png)


## 2. Uso de cluster
### Sin cluster

Posterior a esto haremos la siguiente estructura de carpetas:

```
mkdir /sin-cluster-prueba
```

Entraremos a la carpeta e iniciaremos npm y luego con npm instalaremos express

```
cd sin-cluster-prueba/
npm init -y
npm install express
```

Crearemos el archivo server.js y tendrá este contenido:
```
sudo nano server.js
```

```python
const express = require("express");
const app = express();
const port = 3000;
const limit = 5000000000;

app.get("/", (req, res) => {
  res.send("Hello World!");
});

app.get("/api/:n", function (req, res) {
  let n = parseInt(req.params.n);
  let count = 0;
  if (n > limit) n = limit;
  for (let i = 0; i <= n; i++) {
    count += i;
  }
  res.send(`Final count is ${count}`);
});

app.listen(port, () => {
  console.log(`App listening on port ${port}`);
});
````
![imagenver](../doc/img/cp3.png)

## Pruebas Básicas

![imagenver](../doc/img/cp5.png)

![imagenver](../doc/img/cp2.png)

![imagenver](../doc/img/cp4.png)

### Con cluster

Haremos lo mismo que antes crear carpeta, npm init, npm install express y el server.js con esta configuracion ahora:


![imagenver](../doc/img/cp6.png)

```python
const express = require("express");
const port = 3000;
const limit = 5000000000;
const cluster = require("cluster");
const totalCPUs = require("os").cpus().length;
if (cluster.isMaster) {
  console.log(`Number of CPUs is ${totalCPUs}`);
  console.log(`Master ${process.pid} is running`);
5
  // Fork workers.
  for (let i = 0; i < totalCPUs; i++) {
  cluster.fork();
  }
  cluster.on("exit", (worker, code, signal) => {
  console.log(`worker ${worker.process.pid} died`);
  console.log("Let's fork another worker!");
  cluster.fork();
  });
} else {
  const app = express();
  console.log(`Worker ${process.pid} started`);
  app.get("/", (req, res) => {
  res.send("Hello World!");
  });
  app.get("/api/:n", function (req, res) {
  let n = parseInt(req.params.n);
  let count = 0;
  if (n > limit) n = limit;
  for (let i = 0; i <= n; i++) {
  count += i;
  }
  res.send(`Final count is ${count}`);
  });
  app.listen(port, () => {
  console.log(`App listening on port ${port}`);
  });
}
```

![imagenver](../doc/img/cp7.png)


