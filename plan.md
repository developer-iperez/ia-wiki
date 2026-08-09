Plan de implementación: vault de Obsidian → Quartz → GitHub Pages, gestionado 100% desde Claude Code Cloud (app Android)

Verificado el 2026-08-09 clonando Quartz v5.0.0 real, ejecutando el asistente quartz create --template obsidian y comprobando que npx quartz build compila sin errores. Los comandos de este documento no son teóricos: son los que efectivamente se ejecutaron.

0. Arquitectura final
Tu repo GitHub (rama por defecto, p. ej. "v5")
│
├── content/                  ← tu vault. Aquí escribe Claude Code Cloud
├── quartz/, quartz.config.yaml, quartz.ts   ← motor de Quartz (no tocar)
├── .claude/settings.json     ← permisos acotados para las sesiones cloud
├── CLAUDE.md                 ← instrucciones para esas sesiones
└── .github/workflows/deploy.yml  ← build + publicación automática

Móvil (app Claude, pestaña "Code")
  → sesión cloud sobre el repo → busca/extrae/resume → escribe .md en content/
  → git commit + push
        │
        ▼
GitHub Actions (deploy.yml)
  → npm ci → npx quartz plugin install → npx quartz build → GitHub Pages
        │
        ▼
Navegador del móvil → https://<usuario>.github.io/<repo> (grafo, backlinks, búsqueda)

Nada de esto requiere tener Obsidian instalado en ningún servidor: Quartz lee directamente el Markdown + wikilinks del vault. Obsidian solo entra en juego si además quieres abrir content/ como vault local en tu propio ordenador cuando estés en casa (opcional, ver fase 9).

Fase 1 — Prerrequisitos locales

Necesitas, en el ordenador donde vas a preparar el repo:

Node.js ≥ 22 (node -v)
npm ≥ 10.9 (viene con Node)
Git (git -v)
Una cuenta de GitHub

En Linux, el nodejs de los repositorios del sistema suele ser una versión antigua. Si node -v da menos de 22, instala con nvm o el repositorio oficial de NodeSource.

Fase 2 — Crear el repositorio base (plantilla oficial de Quartz)

Opción recomendada — plantilla de GitHub (evita reconfigurar remotos):

Ve a https://github.com/jackyzha0/quartz, pulsa Use this template → Create a new repository.
Ponle nombre (p. ej. vault, notas, jardin), elige público o privado, Create repository.
Clónalo en local:
bash
   git clone https://github.com/<tu-usuario>/<tu-repo>.git
   cd <tu-repo>

Alternativa (clonar directamente el original): solo si no quieres pasar por GitHub todavía; luego tendrás que apuntar origin a tu propio repo con git remote set-url origin <URL>. No es necesaria si usas la opción anterior.

Fase 3 — Instalar dependencias y ejecutar el asistente
bash
npm ci        # o "npm i" si es la primera instalación

Ejecuta el asistente de creación especificando la plantilla obsidian, que activa el soporte completo de Obsidian Flavored Markdown (wikilinks, callouts, diagramas mermaid) y fija la resolución de enlaces en "shortest" (el mismo comportamiento por defecto de Obsidian), sin preguntarte por ello:

bash
npx quartz create \
  --template obsidian \
  --strategy new \
  --baseUrl "<tu-usuario>.github.io/<tu-repo>"
--strategy new crea content/ vacío — correcto para tu caso, ya que el contenido lo irá generando Claude Code a partir de ahora.
Si ya tienes notas locales que quieras importar, cambia a --strategy copy --source /ruta/a/tu/vault-actual (copia sin tocar el original) o --strategy symlink --source /ruta/... (enlace en vivo).
--baseUrl es crítico: debe ser exactamente <tu-usuario>.github.io/<tu-repo> (sin https://, sin barra final) para que el sitemap, el feed RSS y las URLs limpias funcionen. Si cambias el nombre del repo más adelante, actualízalo en quartz.config.yaml → configuration.baseUrl y en el CNAME que Quartz genera automáticamente en el build.

Salida esperada (la que obtuve en la prueba real):

Created quartz.config.yaml from 'obsidian' template
✓ All configured plugins are already installed
— You're all set! ...

Esto crea/actualiza:

quartz.config.yaml (sustituye cualquier config previa)
content/index.md (portada de ejemplo)
Los ignorePatterns ya vienen con private, templates y .obsidian excluidos del build — útil si más adelante abres content/ como vault local en Obsidian (su carpeta .obsidian/ no se publicará nunca).
Fase 4 — Comprobar que compila en local
bash
npx quartz build --serve

Abre http://localhost:8080. Deberías ver la portada de ejemplo. Ctrl+C para parar. (En la prueba real esto generó 60 ficheros en public/ sin errores; solo avisos benignos sobre fuentes de Google si no hay red.)

Fase 5 — Añadir los ficheros del paquete adjunto

Descarga y descomprime quartz-extras.zip en la raíz de tu repo (se superpone sin sobrescribir nada de Quartz):

bash
unzip quartz-extras.zip -d .

Contiene:

Fichero	Qué hace
.github/workflows/deploy.yml	Construye con npx quartz build y publica en GitHub Pages en cada push.
CLAUDE.md	Instrucciones para que las sesiones de Claude Code sepan dónde y cómo escribir notas.
.claude/settings.json	Permisos acotados: Claude puede leer/escribir en content/ sin pedir confirmación, pero necesita tu aprobación para hacer git push y no puede tocar quartz/, quartz.config.yaml ni .github/.

Antes de hacer commit, abre .github/workflows/deploy.yml y confirma que la rama en on: push: branches: coincide con la tuya:

bash
git branch --show-current

Si tu repo usa main en vez de v5 (lo más probable si tu cuenta de GitHub tiene configurado main como rama por defecto para repos nuevos), cambia esa línea antes de continuar.

Fase 6 — Activar GitHub Pages

En tu repo, en GitHub.com: Settings → Pages → Build and deployment → Source → "GitHub Actions" (no "Deploy from a branch").

Fase 7 — Primer push
bash
git add .
git commit -m "Setup inicial: Quartz + Obsidian + despliegue en GitHub Pages"
git push

Ve a la pestaña Actions de tu repo y comprueba que el workflow "Deploy Quartz site to GitHub Pages" termina en verde. La URL publicada aparecerá en Settings → Pages y también como salida del job de deploy.

Si el Action falla con un error de permisos sobre el entorno github-pages, entra en Settings → Environments, borra el entorno github-pages si ya existe con una configuración antigua, y vuelve a lanzar el workflow (se recrea solo, correctamente).

Fase 8 — Conectar el repo a Claude Code y preparar el entorno cloud
En el móvil, abre la app Claude → pestaña Code (o claude.ai/code desde el navegador la primera vez para el alta).
Conecta GitHub si no lo has hecho, y autoriza el repo que acabas de crear.
Ajusta el acceso de red del entorno cloud, porque por defecto solo permite gestores de paquetes y GitHub, no webs arbitrarias: abre el selector de entorno (icono de nube encima del cuadro de mensaje), edita el entorno (o crea uno nuevo, p. ej. "vault-research") y en Network access cambia de "Trusted" a "Full" — o a "Custom" añadiendo solo los dominios que sueles usar para investigar, si prefieres acotarlo. Sin esto, las búsquedas y extracciones web de Claude pueden fallar o quedarse cortas.
Fase 9 — Flujo de trabajo diario desde el móvil

Desde la pestaña Code, sobre el repo del vault, un prompt típico:

"Busca información actualizada sobre [tema]. Extrae los puntos clave, resume con tus propias palabras y guárdalo como una nota nueva en content/, siguiendo las convenciones de CLAUDE.md. Haz commit."

La sesión corre en la nube: puedes cerrar el móvil y seguirá trabajando. Al terminar, revisa el diff desde la app y aprueba el git push (con las reglas del .claude/settings.json incluido, Claude te pedirá confirmación antes de hacer push). En cuanto el push llega a la rama por defecto, el Action se dispara solo y el sitio se actualiza en 1–2 minutos.

Fase 10 (opcional) — Editar también desde Obsidian en local

Cuando estés en casa, puedes abrir la carpeta content/ (no la raíz del repo) directamente como vault en la app de Obsidian de escritorio. Como los ignorePatterns ya excluyen .obsidian/, puedes editar con total libertad sin que la configuración local de Obsidian contamine el sitio publicado. Haz git pull / git push con tu cliente git habitual para mantener todo sincronizado con lo que vaya escribiendo Claude Code desde el móvil.

Referencia rápida de comandos
Comando	Para qué
npx quartz build --serve	Previsualizar en local (localhost:8080)
npx quartz create -t obsidian ...	(Re)generar quartz.config.yaml desde la plantilla
npx quartz plugin install	Instalar/actualizar plugins según el lockfile
git branch --show-current	Comprobar el nombre real de tu rama por defecto
Troubleshooting rápido
El Action falla en "Deploy to GitHub Pages" por permisos de entorno → borra el entorno github-pages en Settings → Environments y relanza.
Las notas no aparecen aunque el Action fue verde → revisa que no tengan draft: true en el frontmatter y que no estén dentro de content/private/ o content/templates/ (excluidas a propósito).
Los enlaces [[nota]] no resuelven → confirma que quartz.config.yaml sigue con markdownLinkResolution: shortest (lo pone la plantilla obsidian por defecto; no debería tocarse).
Claude Code no puede buscar en la web desde el móvil → revisa la Fase 8, el acceso de red del entorno cloud.