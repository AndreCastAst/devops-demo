# DevOps Demo

Un ejemplo **mínimo y completo** del ciclo DevOps: desde escribir código hasta verlo vivo en producción, con automatización que impide que código roto llegue al usuario.

---

## ¿Qué es esto?

Este repositorio demuestra el **lazo cerrado** de DevOps en su forma más simple:

> "Hago push → el CI prueba y construye el contenedor automáticamente → si todo pasa, Render despliega a producción solo → la URL pública refleja el cambio."

No es una app compleja. Es una demostración de que **el proceso importa más que el código**.

---

## Diagrama del pipeline (loop, no línea recta)

```mermaid
flowchart LR
    A[Escribo código] --> B[git push]
    B --> C{CI: pytest + docker build}
    C -->|falla| A
    C -->|pasa| D[CD: Render deploy]
    D --> E[URL pública]
    E --> F[/health · logs]
    F -.feedback.-> A
```

La flecha de feedback es la clave: producción le habla de vuelta al desarrollador.

---

## Estructura del proyecto

```
devops-demo/
├── app.py                  # Flask: endpoints / y /health
├── requirements.txt        # flask + pytest
├── test_app.py             # Tests reales con pytest
├── Dockerfile              # Empaqueta la app
├── .dockerignore
├── .gitignore
├── README.md
└── .github/
    └── workflows/
        └── ci.yml          # Test + build en cada push
```

---

## Cómo correr localmente

### Sin Docker

```bash
python -m venv venv
# Windows:
venv\Scripts\activate
# macOS/Linux:
source venv/bin/activate

pip install -r requirements.txt
python app.py
# Abre: http://localhost:5000
# Verifica: http://localhost:5000/health
```

### Correr los tests

```bash
pytest -v
```

### Con Docker

```bash
docker build -t devops-demo .
docker run -p 5000:5000 devops-demo
# Verifica: http://localhost:5000/health
```

---

## Cómo está desplegado (Render)

1. En [render.com](https://render.com) → **New → Web Service**
2. Conectar este repositorio de GitHub
3. Render detecta el `Dockerfile` automáticamente (Runtime: Docker)
4. Plan: **Free**
5. Click **Deploy**
6. La app queda viva en `https://<tu-app>.onrender.com`

**Auto-deploy activado:** cada `git push` a `main` redespliega solo. Sin tocar servidores.

> **Nota cold start (free tier):** el servicio se duerme tras 15 min sin tráfico. El primer request puede tardar 30–60 s. Es normal, no es un bug.

---

## Demo 1 — Romper el build a propósito (CI en rojo)

Esta demo muestra **por qué existe el CI**: bloquear que código roto llegue a producción.

```bash
# 1. Crear una rama de prueba
git checkout -b demo/break-ci

# 2. Editar app.py: cambiar el return de /health a código 500
#    return jsonify({"status": "broken"}), 500

# 3. Push
git push origin demo/break-ci
```

4. Ir a la pestaña **Actions** en GitHub → el workflow aparece en **ROJO**
5. El error es: el test `test_health_returns_200` falló porque recibió 500
6. Render **no** redesplegó porque el CI falló — producción está intacta

```bash
# 7. Revertir
git checkout main
git push origin --delete demo/break-ci
```

**Punto de la demo:** el CI es la barrera. Sin él, el error llega al usuario.

---

## Demo 2 — Cerrar el loop (push → producción cambia sola)

Esta demo muestra el **lazo cerrado** funcionando end-to-end.

```bash
# 1. Cambiar el mensaje del endpoint /
#    En app.py: return "DevOps Demo v2 — el loop está cerrado ✅"

git add app.py
git commit -m "feat: actualizar mensaje del índice"
git push origin main
```

2. Ir a **Actions** → el workflow corre y termina en **VERDE**
3. Render recibe el evento y redespliega automáticamente (1–2 min)
4. Abrir `https://<tu-app>.onrender.com` → el nuevo mensaje ya está vivo

**Punto de la demo:** nadie tocó un servidor. El push fue suficiente.

---

## Endpoints

| Endpoint | Descripción |
|---|---|
| `GET /` | Mensaje de bienvenida |
| `GET /health` | `{"status": "ok"}` — punto de observabilidad |
