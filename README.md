# Estruturas de Proyectos Express-Generator + MVC - de Básico a Pro

Refactorizando app

## 🧱 Proyecto base creado con `express-generator`

Cuando corrés este comando:

```bash
npx express-generator myapp --view=ejs
```

Y luego:

```bash
cd myapp
npm install
```

Se genera automáticamente esta estructura:

```
📁 myapp/
├── 📁 bin/
│   └── 📄 www                      ← PUNTO DE ENTRADA DEL SERVIDOR
├── 📁 public/
│   ├── 📁 images/
│   ├── 📁 javascripts/
│   └── 📁 stylesheets/
│       └── 📄 style.css
├── 📁 routes/
│   ├── 📄 index.js
│   └── 📄 users.js
├── 📁 views/
│   ├── 📄 error.ejs
│   ├── 📄 index.ejs
│   └── 📄 layout.ejs
├── 📄 app.js                       ← CONFIGURACIÓN PRINCIPAL DE EXPRESS
├── 📄 package.json
└── 📄 package-lock.json
```

---

### 📁 Estructura explicada

| Carpeta / Archivo | Función                                                                       |
| ----------------- | ----------------------------------------------------------------------------- |
| `bin/www`         | Archivo que inicia el servidor HTTP (`http.createServer`) y llama a `app.js`. |
| `app.js`          | Configura middlewares, rutas, vistas, etc. Se importa desde `bin/www`.        |
| `routes/`         | Archivos donde se definen las rutas Express.                                  |
| `views/`          | Vistas con EJS por defecto (`index.ejs`, `layout.ejs`, `error.ejs`).          |
| `public/`         | Archivos estáticos como CSS, imágenes o JS del lado cliente.                  |
| `package.json`    | Scripts, dependencias y configuración del proyecto.                           |

---

### 📦 Contenido básico de los archivos clave

#### 📄 `bin/www`

```js
#!/usr/bin/env node

const app = require('../app');
const http = require('http');

const port = normalizePort(process.env.PORT || '3000');
app.set('port', port);

const server = http.createServer(app);
server.listen(port);

function normalizePort(val) {
  const port = parseInt(val, 10);
  if (isNaN(port)) return val;
  if (port >= 0) return port;
  return false;
}
```

---

#### 📄 `app.js`

```js
const express = require('express');
const path = require('path');
const cookieParser = require('cookie-parser');
const logger = require('morgan');

const indexRouter = require('./routes/index');
const usersRouter = require('./routes/users');

const app = express();

app.set('views', path.join(__dirname, 'views'));
app.set('view engine', 'ejs');

app.use(logger('dev'));
app.use(express.json());
app.use(express.urlencoded({ extended: false }));
app.use(cookieParser());
app.use(express.static(path.join(__dirname, 'public')));

app.use('/', indexRouter);
app.use('/users', usersRouter);

module.exports = app;
```

---

#### 📄 `routes/index.js`

```js
const express = require('express');
const router = express.Router();

router.get('/', function(req, res) {
  res.render('index', { title: 'Express' });
});

module.exports = router;
```

---

#### 📄 `views/index.ejs`

```html
<!DOCTYPE html>
<html>
<head>
  <title><%= title %></title>
  <link rel="stylesheet" href="/stylesheets/style.css" />
</head>
<body>
  <h1><%= title %></h1>
  <p>Welcome to <%= title %></p>
</body>
</html>
```

---

### 📌 Conclusión

El `express-generator` crea una estructura **funcional pero monolítica** (todo en la raíz), ideal para prototipos rápidos. Luego, podés reorganizarla como vimos en la estructura profesional que te recomendé antes (con `/src`, `controllers/`, `services/`, etc.).

---

## 🧱 Estructura Profesional Moderna (Express + MVC + Clean Architecture)

Proyecto con Express.js, organizado siguiendo el patrón **MVC** + separaciones de lógica por responsabilidad. Está pensada para **escalar**, **testear**, y **mantener** fácilmente.

```
📁 myapp/
├── 📁 src/
│   ├── 📁 config/              ← Configuraciones generales (DB, variables, etc.)
│   │   └── 📄 index.js
│   ├── 📁 controllers/         ← Controladores: lógica de control de rutas
│   │   └── 📄 user.controller.js
│   ├── 📁 models/              ← Modelos (ej. con Mongoose, Sequelize o clases simples)
│   │   └── 📄 user.model.js
│   ├── 📁 routes/              ← Definición de rutas y vinculación con controladores
│   │   └── 📄 user.routes.js
│   ├── 📁 services/            ← Lógica de negocio: validaciones, reglas, etc.
│   │   └── 📄 user.service.js
│   ├── 📁 middlewares/         ← Middlewares personalizados (auth, logger, etc.)
│   │   └── 📄 auth.middleware.js
│   ├── 📁 utils/               ← Funciones auxiliares (helpers, logger, etc.)
│   │   └── 📄 logger.js
│   ├── 📁 views/               ← Vistas EJS, Pug, Handlebars, etc.
│   │   ├── 📄 index.ejs
│   │   └── 📄 layout.ejs
│   └── 📄 app.js               ← Configura Express, middlewares y rutas
├── 📁 public/                  ← Archivos estáticos (CSS, JS del cliente, imgs)
│   └── 📁 stylesheets/
│       └── 📄 style.css
├── 📄 server.js                ← Punto de entrada principal del servidor
├── 📄 .env                     ← Variables de entorno
├── 📄 .gitignore
├── 📄 package.json
└── 📄 README.md
```

---

### 📦 Contenido típico de los archivos clave

---

#### 📄 `server.js`

```js
import http from 'http';
import app from './src/app.js';
import dotenv from 'dotenv';

dotenv.config();

const PORT = process.env.PORT || 3000;

const server = http.createServer(app);

server.listen(PORT, () => {
  console.log(`Servidor escuchando en http://localhost:${PORT}`);
});
```

---

#### 📄 `src/app.js`

```js
import express from 'express';
import path from 'path';
import cookieParser from 'cookie-parser';
import logger from 'morgan';

import userRoutes from './routes/user.routes.js';

const app = express();

// Configuración de vistas
app.set('views', path.resolve('src', 'views'));
app.set('view engine', 'ejs');

// Middlewares
app.use(logger('dev'));
app.use(express.json());
app.use(express.urlencoded({ extended: false }));
app.use(cookieParser());
app.use(express.static('public'));

// Rutas
app.use('/users', userRoutes);

// Ruta raíz para prueba
app.get('/', (req, res) => {
  res.render('index', { title: 'Express App Moderna' });
});

export default app;
```

---

#### 📄 `src/routes/user.routes.js`

```js
import express from 'express';
import { getAllUsers } from '../controllers/user.controller.js';

const router = express.Router();

router.get('/', getAllUsers);

export default router;
```

---

#### 📄 `src/controllers/user.controller.js`

```js
import { fetchUsers } from '../services/user.service.js';

export const getAllUsers = (req, res) => {
  const users = fetchUsers();
  res.json(users);
};
```

---

#### 📄 `src/services/user.service.js`

```js
export const fetchUsers = () => {
  return [
    { id: 1, name: 'Dante' },
    { id: 2, name: 'Alda' },
  ];
};
```

---

#### 📄 `src/views/index.ejs`

```html
<!DOCTYPE html>
<html>
<head>
  <title><%= title %></title>
  <link rel="stylesheet" href="/stylesheets/style.css" />
</head>
<body>
  <h1><%= title %></h1>
  <p>¡Bienvenido a tu app profesional Express!</p>
</body>
</html>
```

---

### 🎯 Ventajas de esta estructura

✅ Escalable: podés agregar fácilmente nuevas funcionalidades.

✅ Separación de responsabilidades: cada capa tiene su función.

✅ Legible y mantenible.

✅ Ideal para testing automatizado.

✅ Facilita la integración de bases de datos, autenticación, logging, etc.

---

### 💡 Pro tip - Comando --watch

Agregá a tu `package.json`:

```json
"scripts": {
  "start": "node server.js",
  "dev": "node --watch server.js"
}
```
Para desarrollo sin usar `nodemon`.

Luego en consola : >_ 
```  npm run dev ``` 