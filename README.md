# VocLab

[English](#english) | [Español](#español)

---

# English

## Overview

**VocLab** is a web application for creating, validating, managing, visualizing and exporting controlled vocabularies. It is designed for educational, research and professional contexts where users need to work with vocabularies, taxonomies, thesauri or SKOS-like structures through a simple tabular workflow.

VocLab allows users to upload vocabularies from CSV or TSV files, validate their structure and semantic consistency, browse their content through different views, inspect vocabulary statistics, visualize concept relationships as a graph, and export valid vocabularies in interoperable formats such as CSV, TBL, Turtle and RDF/XML.

The application is especially useful for teaching controlled vocabularies, metadata, taxonomies, thesauri and semantic web principles without requiring users to edit RDF directly.

![VocLab dashboard](assets/img/image-1.png)

*Figure 1. VocLab dashboard and vocabulary catalogue.*

---

## Main features

VocLab provides the following main features:

* User authentication and role-based access control.
* Initial administrator password setup.
* User management by administrators.
* Vocabulary creation from CSV or TSV files.
* Vocabulary metadata management.
* Vocabulary validation according to a predefined tabular template.
* Support for multilingual labels and notes.
* Support for concept schemes, concepts and collections.
* Validation of hierarchical, associative and collection membership relations.
* Detection of duplicate codes, invalid language tags and hierarchical cycles.
* Public or private visibility for vocabularies.
* Interactive vocabulary browsing.
* Alphabetical, hierarchical and collection-based views.
* Graph visualization of vocabulary relationships.
* Vocabulary statistics and per-concept indicators.
* Export to CSV, TBL, Turtle and RDF/XML.
* English and Spanish interface.

---

## Requirements

VocLab is a PHP and MySQL/MariaDB web application.

Recommended requirements:

* PHP 8.0 or higher.
* MySQL or MariaDB.
* A web server with PHP support, such as Apache HTTP Server or Nginx.
* PDO MySQL extension enabled.
* A modern web browser.

The application uses the following front-end libraries:

* Bootstrap.
* Bootstrap Icons.
* Tabulator.
* D3.js.

Local fallback copies of these libraries may be included under `assets/vendor/`.

---

## Repository structure

A typical VocLab installation contains the following folders and files:

```text
voclab/
├── app/
│   ├── auth.php
│   ├── db.php
│   ├── i18n.php
│   ├── language_selector.php
│   ├── layout.php
│   ├── rdf_exporter.php
│   ├── users.php
│   ├── validator.php
│   ├── vocabularies.php
│   ├── vocabulary_parser.php
│   ├── vocabulary_table_editor.php
│   └── vocabulary_utils.php
├── assets/
│   ├── css/
│   ├── img/
│   ├── js/
│   └── vendor/
├── config/
│   └── config.php
├── examples/
├── lang/
│   ├── en.php
│   └── es.php
├── sql/
│   └── schema.sql
├── index.php
├── login.php
├── logout.php
├── setup_admin.php
├── new_vocabulary.php
├── edit_vocabulary.php
├── view_vocabulary.php
├── export_vocabulary.php
├── delete_vocabulary.php
├── manage_users.php
├── edit_user.php
├── delete_user.php
├── change_language.php
├── README.md
└── LICENSE
```

---

## Installation

### 1. Copy the application files

Clone or download the repository into your web server directory.

```bash
git clone https://github.com/your-user/voclab.git
```

Place the project in the directory served by your web server, for example:

```text
/var/www/html/voclab
```

or another equivalent location depending on your server configuration.

---

### 2. Create the database

Create an empty MySQL or MariaDB database for VocLab:

```sql
CREATE DATABASE voclab CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
```

Create or select a database user with the required permissions:

```sql
CREATE USER 'voclab_user'@'localhost' IDENTIFIED BY 'change_this_password';
GRANT ALL PRIVILEGES ON voclab.* TO 'voclab_user'@'localhost';
FLUSH PRIVILEGES;
```

---

### 3. Configure the application

Edit:

```text
config/config.php
```

and update the database connection values:

```php
return [
	'db_host' => 'localhost',
	'db_name' => 'voclab',
	'db_user' => 'voclab_user',
	'db_pass' => 'change_this_password',
	'db_charset' => 'utf8mb4',

	'app_name' => 'VocLab',
	'app_url' => 'http://localhost/voclab'
];
```

Before deploying VocLab in a production environment, replace all example credentials with secure values.

---

### 4. Initialize the database schema

VocLab can initialize the database schema automatically from:

```text
sql/schema.sql
```

when the application is first executed and the required tables are not found.

Alternatively, you can import the schema manually:

```bash
mysql -u voclab_user -p voclab < sql/schema.sql
```

---

### 5. Open the application

Open the application in your browser:

```text
http://localhost/voclab
```

On first execution, VocLab checks whether the default administrator account exists and whether its password is still pending. If so, the application redirects to the initial administrator setup page.

---

## First access

VocLab includes an initial administrator account created by the database schema.

The default administrator username is:

```text
admin
```

The password is intentionally empty in the initial schema. On first execution, VocLab displays a setup page where the administrator must define the initial password.

After that, the administrator can log in and manage users and vocabularies.

---

## User management

Only administrator users can manage user accounts.

From the user management page, administrators can:

* View existing users.
* Create users in batch mode.
* Edit user profile data.
* Assign user roles.
* Set or update user passwords.
* Delete users when they have no associated vocabularies.

VocLab supports two basic roles:

```text
admin
user
```

Administrators can manage all vocabularies and all users. Regular users can manage only the vocabularies assigned to them.

![User management](assets/img/image-2.png)

*Figure 2. User management interface.*

---

## Vocabulary management

Authenticated users can create vocabularies from CSV or TSV files.

Each vocabulary includes basic metadata such as:

* Title.
* Description.
* Base URI.
* Optional prefix.
* Available languages.
* Source file.
* Public or private visibility.

A vocabulary can be marked as public only when it is valid. Public vocabularies can be accessed without login, but public access does not grant editing or deletion permissions.

![New vocabulary form](assets/img/image-3.png)

*Figure 3. Vocabulary creation form.*

---

## Vocabulary source format

VocLab uses a tabular source format to represent controlled vocabularies. Vocabulary files can be uploaded as **CSV** or **TSV** files. Each row represents one vocabulary resource, and each column describes a specific property of that resource.

The first row of the file must contain the column names. VocLab expects the following columns, in this order:

```text
resource_type,code,prefLabel,altLabel,hiddenLabel,notation,definition,scopeNote,broader,related,memberOf,sameAs
```

### Columns

| Column          | Required | Description                                                                                                                                    |
| --------------- | -------: | ---------------------------------------------------------------------------------------------------------------------------------------------- |
| `resource_type` |      Yes | Type of resource represented by the row. Supported values are `concept`, `collection` and `conceptscheme`.                                     |
| `code`          |      Yes | Local identifier of the resource. It must be unique within the vocabulary. Other relationship columns use this value to refer to the resource. |
| `prefLabel`     |      Yes | Preferred label of the resource. Values must include language tags using the `value@language` syntax.                                          |
| `altLabel`      |       No | Alternative labels or synonyms. Multiple values can be separated with `\|`.                                                                    |
| `hiddenLabel`   |       No | Hidden labels, spelling variants or non-preferred access terms that should not be displayed as ordinary synonyms.                              |
| `notation`      |       No | Classification code, notation or internal reference associated with the resource.                                                              |
| `definition`    |       No | Definition of the resource. Values should use the `value@language` syntax.                                                                     |
| `scopeNote`     |       No | Scope note explaining how the resource should be used. Values should use the `value@language` syntax.                                          |
| `broader`       |       No | Code of one or more broader concepts. Multiple broader concepts can be separated with `\|`.                                                    |
| `related`       |       No | Code of one or more associated concepts. Multiple related concepts can be separated with `\|`.                                                 |
| `memberOf`      |       No | Code of one or more collections to which the resource belongs. Multiple collections can be separated with `\|`.                                |
| `sameAs`        |       No | External URI, IRI or URN identifying an equivalent resource. Multiple values can be separated with `\|`.                                       |

### Resource types

The `resource_type` column accepts three values:

```text
concept
collection
conceptscheme
```

A `concept` represents a vocabulary term or category. Most rows in a vocabulary will normally be concepts.

A `collection` represents a grouping of resources. Collections are useful when a set of concepts needs to be grouped without creating a hierarchical broader/narrower relation.

A `conceptscheme` represents the vocabulary as a whole. A vocabulary may include a row of this type to provide labels, definitions or notes for the complete concept scheme.

### Multilingual values

VocLab represents multilingual labels and notes using the following syntax:

```text
value@language
```

For example:

```text
Climate change@en
```

A field can contain several multilingual values separated with the pipe character:

```text
Climate change@en|Cambio climático@es
```

The language code must match one of the languages configured for the vocabulary. For example, if the vocabulary languages are:

```text
en,es
```

then values such as the following are valid:

```text
Action@en|Acción@es
```

but a value such as the following would be invalid if `fr` is not one of the available languages:

```text
Action@en|Acción@es|Action culturelle@fr
```

The following columns use multilingual values:

```text
prefLabel
altLabel
hiddenLabel
definition
scopeNote
```

### Relationships between resources

Relationships are expressed by writing the `code` of another resource.

For example, if the vocabulary contains this concept:

```csv
concept,action,Action@en|Acción@es,,,,,,,,,
```

another concept can refer to it as a broader concept:

```csv
concept,platform-games,Platform games@en|Juegos de plataformas@es,,,,,,action,,,
```

Multiple relationship targets are separated with `|`:

```csv
concept,metroidvania,Metroidvania@en|Metroidvania@es,,,,,,platform-games,open-world|action-rpg,,
```

VocLab validates that the referenced codes exist in the vocabulary.

The main relationship columns are:

| Column     | Meaning                                        |
| ---------- | ---------------------------------------------- |
| `broader`  | Hierarchical parent concept.                   |
| `related`  | Associative relationship with another concept. |
| `memberOf` | Collection to which the resource belongs.      |
| `sameAs`   | External equivalent URI, IRI or URN.           |

### Example

The following simplified example shows a small bilingual vocabulary fragment:

```csv
resource_type,code,prefLabel,altLabel,hiddenLabel,notation,definition,scopeNote,broader,related,memberOf,sameAs
conceptscheme,video-game-genres,Video game genres@en|Géneros de videojuegos@es,,,,A vocabulary for describing video game genres.@en|Vocabulario para describir géneros de videojuegos.@es,,,,,
concept,action,Action@en|Acción@es,Action game@en|Videojuego de acción@es,,GEN-ACT,A genre focused on real-time challenges and player reaction.@en|Género centrado en desafíos en tiempo real y reacción del jugador.@es,,,,,
concept,platform-games,Platform games@en|Juegos de plataformas@es,Platformers@en,,GEN-PLA,A genre based on jumping movement and spatial progression.@en|Género basado en saltos movimiento y progresión espacial.@es,,action,,,
concept,metroidvania,Metroidvania@en|Metroidvania@es,,,GEN-MET,A subgenre based on interconnected exploration and progressive unlocking of abilities.@en|Subgénero basado en exploración interconectada y desbloqueo progresivo de habilidades.@es,,platform-games,action-rpg,,
collection,action-subgenres,Action subgenres@en|Subgéneros de acción@es,,,,,,,,
concept,action-rpg,Action RPG@en|RPG de acción@es,,,GEN-ARPG,A role-playing game with real-time action mechanics.@en|Juego de rol con mecánicas de acción en tiempo real.@es,,action,,action-subgenres,
```

In this example:

* `video-game-genres` is the concept scheme.
* `action`, `platform-games`, `metroidvania` and `action-rpg` are concepts.
* `platform-games` has `action` as its broader concept.
* `metroidvania` has `platform-games` as its broader concept.
* `metroidvania` is related to `action-rpg`.
* `action-rpg` belongs to the `action-subgenres` collection.

### Validation rules

When a vocabulary is uploaded or edited, VocLab checks that:

* the required columns exist and appear in the expected order;
* each resource has a valid `resource_type`;
* each resource has a unique `code`;
* each resource has at least one valid `prefLabel`;
* multilingual values use valid language tags;
* language tags match the available languages configured for the vocabulary;
* relationship targets in `broader`, `related` and `memberOf` exist;
* broader relations do not create hierarchical cycles;
* related concepts are not placed in the same direct hierarchical path;
* values in `sameAs` are valid URI, IRI or URN values.

---

## Vocabulary validation

When a vocabulary is uploaded or edited, VocLab validates its content before storing or updating it.

The validation process checks, among other aspects:

* Expected column structure.
* Required values.
* Valid resource types.
* Duplicate resource codes.
* Valid language-tagged values.
* Use of available vocabulary languages.
* Valid broader, narrower and related references.
* Valid collection membership references.
* Hierarchical cycles.
* Related concepts placed in the same direct hierarchical path.
* Valid URI, IRI or URN values.

If validation fails, VocLab reports the detected errors so the user can correct the source file or the editable table.

---

## Editing vocabularies

Existing vocabularies can be edited by authorized users.

The editing page allows users to:

* Update vocabulary metadata.
* Change public/private visibility.
* Replace the source CSV or TSV file.
* Edit the vocabulary table directly through a grid interface.
* Revalidate the vocabulary after changes.

The grid editor provides a spreadsheet-like interface for editing vocabulary rows and columns without leaving the application.

![Vocabulary table editor](assets/img/image-4.png)

*Figure 4. Spreadsheet-like vocabulary table editor.*

---

## Using vocabularies

The vocabulary detail page provides several ways to explore a vocabulary.

Users can browse vocabulary resources through:

* Alphabetical view.
* Hierarchical view.
* Group or collection-based view.
* Search box.
* Resource detail panel.
* Internal links between related resources.

The detail panel displays labels, notes, semantic relations, collection membership, mappings and external URI references.

![Vocabulary browser](assets/img/image-5.png)

*Figure 5. Interactive vocabulary browser.*

---

## Graph visualization

VocLab includes an interactive graph visualization based on D3.js.

The graph view allows users to:

* Explore concept hierarchies visually.
* Display broader and narrower relations.
* Display related concept links.
* Expand or collapse concept branches.
* Drag nodes.
* Zoom and pan the graph.
* Open a selected resource in the vocabulary viewer.
* Export the current graph as SVG.

![Vocabulary graph](assets/img/image-6.png)

*Figure 6. Interactive vocabulary graph visualization.*

---

## Vocabulary statistics

VocLab can calculate descriptive statistics for a vocabulary.

Statistics include:

* Number of concepts.
* Number of collections.
* Number of concept schemes.
* Number of labels.
* Number of definitions and scope notes.
* Number of semantic relations.
* Number of mapping relations.
* Number of root, intermediate, final and orphan concepts.
* Number of ambiguous concepts.
* Hierarchical depth indicators.
* Branch size indicators.

Global statistics and per-concept statistics can be downloaded as CSV files.

![Vocabulary statistics](assets/img/image-7.png)

*Figure 7. Vocabulary statistics modal.*

---

## Export options

Valid vocabularies can be exported in several formats:

```text
CSV
TBL
Turtle
RDF/XML
```

CSV and TBL exports are generated from the stored source table.

Turtle and RDF/XML exports are generated from the internal vocabulary model and include SKOS-compatible RDF statements for:

* Concept schemes.
* Concepts.
* Collections.
* Preferred labels.
* Alternative labels.
* Hidden labels.
* Definitions.
* Scope notes.
* Broader and narrower relations.
* Related relations.
* Collection members.
* Top concepts.
* Mapping links.
* `owl:sameAs` links.

![Vocabulary export options](assets/img/image-8.png)

*Figure 8. RDF and tabular export options.*

---

## Public vocabularies

A vocabulary can be made public when it has passed validation.

Public vocabularies are visible to anonymous visitors from the public catalogue. Anonymous users can browse and export public vocabularies but cannot edit or delete them.

Private vocabularies are visible only to administrators and assigned users.

---

## Example files

The repository may include example CSV files that can be used to test the application.

Suggested location:

```text
examples/
```

Possible examples:

```text
voclab_sample.csv
voclab_sample_errors.csv
voclab_ipct-mediatopic.csv
```

These files illustrate the expected vocabulary template and can be used for testing validation, error reporting and vocabulary visualization.

---

## Security notes

Before deploying VocLab in production:

* Replace example database credentials in `config/config.php`.
* Use a strong administrator password.
* Restrict database permissions to the required database.
* Serve the application over HTTPS.
* Keep PHP, MySQL/MariaDB and the web server updated.
* Avoid uploading development files, drafts or database backups to the public server.

---

## License

VocLab is free software released under the **GNU General Public License v3.0**.

You may redistribute and/or modify it under the terms of the GNU General Public License as published by the Free Software Foundation, either version 3 of the License, or any later version.

See the `LICENSE` file for details.

---

# Español

## Descripción general

**VocLab** es una aplicación web para crear, validar, gestionar, visualizar y exportar vocabularios controlados. Está pensada para contextos docentes, de investigación y profesionales en los que sea necesario trabajar con vocabularios, taxonomías, tesauros o estructuras similares a SKOS mediante un flujo de trabajo sencillo basado en tablas.

VocLab permite cargar vocabularios desde ficheros CSV o TSV, validar su estructura y consistencia semántica, navegar por sus contenidos mediante distintas vistas, consultar estadísticas, visualizar las relaciones entre conceptos como un grafo y exportar vocabularios válidos en formatos interoperables como CSV, TBL, Turtle y RDF/XML.

La aplicación resulta especialmente útil para la enseñanza de vocabularios controlados, metadatos, taxonomías, tesauros y principios de la web semántica sin necesidad de que los usuarios editen RDF directamente.

![Panel principal de VocLab](assets/img/image-1.png)

*Figura 1. Panel principal de VocLab y catálogo de vocabularios.*

---

## Funcionalidades principales

VocLab ofrece las siguientes funcionalidades:

* Autenticación de usuarios y control de acceso basado en roles.
* Configuración inicial de la cuenta administradora.
* Gestión de usuarios por parte de administradores.
* Creación de vocabularios a partir de ficheros CSV o TSV.
* Gestión de metadatos de vocabularios.
* Validación de vocabularios según una plantilla tabular predefinida.
* Soporte para etiquetas y notas multilingües.
* Soporte para esquemas de conceptos, conceptos y colecciones.
* Validación de relaciones jerárquicas, asociativas y de pertenencia a colecciones.
* Detección de códigos duplicados, etiquetas de idioma no válidas y ciclos jerárquicos.
* Visibilidad pública o privada de vocabularios.
* Navegación interactiva por vocabularios.
* Vistas alfabética, jerárquica y basada en colecciones.
* Visualización gráfica de relaciones entre conceptos.
* Estadísticas del vocabulario e indicadores por concepto.
* Exportación a CSV, TBL, Turtle y RDF/XML.
* Interfaz en inglés y español.

---

## Requisitos

VocLab es una aplicación web desarrollada en PHP y MySQL/MariaDB.

Requisitos recomendados:

* PHP 8.0 o superior.
* MySQL o MariaDB.
* Servidor web con soporte PHP, como Apache HTTP Server o Nginx.
* Extensión PDO MySQL habilitada.
* Navegador web moderno.

La aplicación utiliza las siguientes bibliotecas de interfaz:

* Bootstrap.
* Bootstrap Icons.
* Tabulator.
* D3.js.

Pueden incluirse copias locales de respaldo de estas bibliotecas en `assets/vendor/`.

---

## Estructura del repositorio

Una instalación típica de VocLab contiene las siguientes carpetas y ficheros:

```text
voclab/
├── app/
│   ├── auth.php
│   ├── db.php
│   ├── i18n.php
│   ├── language_selector.php
│   ├── layout.php
│   ├── rdf_exporter.php
│   ├── users.php
│   ├── validator.php
│   ├── vocabularies.php
│   ├── vocabulary_parser.php
│   ├── vocabulary_table_editor.php
│   └── vocabulary_utils.php
├── assets/
│   ├── css/
│   ├── img/
│   ├── js/
│   └── vendor/
├── config/
│   └── config.php
├── examples/
├── lang/
│   ├── en.php
│   └── es.php
├── sql/
│   └── schema.sql
├── index.php
├── login.php
├── logout.php
├── setup_admin.php
├── new_vocabulary.php
├── edit_vocabulary.php
├── view_vocabulary.php
├── export_vocabulary.php
├── delete_vocabulary.php
├── manage_users.php
├── edit_user.php
├── delete_user.php
├── change_language.php
├── README.md
└── LICENSE
```

---

## Instalación

### 1. Copiar los ficheros de la aplicación

Clona o descarga el repositorio en el directorio de tu servidor web.

```bash
git clone https://github.com/your-user/voclab.git
```

Coloca el proyecto en el directorio servido por tu servidor web, por ejemplo:

```text
/var/www/html/voclab
```

o en otra ubicación equivalente según la configuración del servidor.

---

### 2. Crear la base de datos

Crea una base de datos vacía en MySQL o MariaDB para VocLab:

```sql
CREATE DATABASE voclab CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
```

Crea o selecciona un usuario de base de datos con los permisos necesarios:

```sql
CREATE USER 'voclab_user'@'localhost' IDENTIFIED BY 'change_this_password';
GRANT ALL PRIVILEGES ON voclab.* TO 'voclab_user'@'localhost';
FLUSH PRIVILEGES;
```

---

### 3. Configurar la aplicación

Edita:

```text
config/config.php
```

y actualiza los valores de conexión a la base de datos:

```php
return [
	'db_host' => 'localhost',
	'db_name' => 'voclab',
	'db_user' => 'voclab_user',
	'db_pass' => 'change_this_password',
	'db_charset' => 'utf8mb4',

	'app_name' => 'VocLab',
	'app_url' => 'http://localhost/voclab'
];
```

Antes de desplegar VocLab en un entorno de producción, sustituye todas las credenciales de ejemplo por valores seguros.

---

### 4. Inicializar el esquema de la base de datos

VocLab puede inicializar automáticamente el esquema de la base de datos a partir de:

```text
sql/schema.sql
```

cuando la aplicación se ejecuta por primera vez y no encuentra las tablas necesarias.

También puedes importar el esquema manualmente:

```bash
mysql -u voclab_user -p voclab < sql/schema.sql
```

---

### 5. Abrir la aplicación

Abre la aplicación en el navegador:

```text
http://localhost/voclab
```

En la primera ejecución, VocLab comprueba si existe la cuenta administradora por defecto y si su contraseña está todavía pendiente. Si es así, redirige a la página de configuración inicial del administrador.

---

## Primer acceso

VocLab incluye una cuenta administradora inicial creada por el esquema de base de datos.

El nombre de usuario administrador por defecto es:

```text
admin
```

La contraseña se deja intencionadamente vacía en el esquema inicial. En la primera ejecución, VocLab muestra una página de configuración donde el administrador debe definir la contraseña inicial.

Después, el administrador podrá iniciar sesión y gestionar usuarios y vocabularios.

---

## Gestión de usuarios

Solo los usuarios administradores pueden gestionar cuentas de usuario.

Desde la página de gestión de usuarios, los administradores pueden:

* Ver los usuarios existentes.
* Crear usuarios en modo por lotes.
* Editar los datos de perfil de un usuario.
* Asignar roles.
* Establecer o actualizar contraseñas.
* Eliminar usuarios cuando no tienen vocabularios asociados.

VocLab soporta dos roles básicos:

```text
admin
user
```

Los administradores pueden gestionar todos los vocabularios y todos los usuarios. Los usuarios normales solo pueden gestionar los vocabularios que tienen asignados.

![Gestión de usuarios](assets/img/image-2.png)

*Figura 2. Interfaz de gestión de usuarios.*

---

## Gestión de vocabularios

Los usuarios autenticados pueden crear vocabularios a partir de ficheros CSV o TSV.

Cada vocabulario incluye metadatos básicos como:

* Título.
* Descripción.
* URI base.
* Prefijo opcional.
* Idiomas disponibles.
* Fichero fuente.
* Visibilidad pública o privada.

Un vocabulario puede marcarse como público solo cuando es válido. Los vocabularios públicos pueden consultarse sin iniciar sesión, pero el acceso público no concede permisos de edición ni eliminación.

![Formulario de nuevo vocabulario](assets/img/image-3.png)

*Figura 3. Formulario de creación de vocabularios.*

---

## Formato fuente de los vocabularios

VocLab utiliza un formato tabular para representar vocabularios controlados. Los vocabularios pueden cargarse como ficheros **CSV** o **TSV**. Cada fila representa un recurso del vocabulario, y cada columna describe una propiedad concreta de ese recurso.

La primera fila del fichero debe contener los nombres de las columnas. VocLab espera las siguientes columnas, en este orden:

```text
resource_type,code,prefLabel,altLabel,hiddenLabel,notation,definition,scopeNote,broader,related,memberOf,sameAs
```

### Columnas

| Columna         | Obligatoria | Descripción                                                                                                                                   |
| --------------- | ----------: | --------------------------------------------------------------------------------------------------------------------------------------------- |
| `resource_type` |          Sí | Tipo de recurso representado por la fila. Los valores admitidos son `concept`, `collection` y `conceptscheme`.                                |
| `code`          |          Sí | Identificador local del recurso. Debe ser único dentro del vocabulario. Las columnas de relaciones usan este valor para referirse al recurso. |
| `prefLabel`     |          Sí | Etiqueta preferente del recurso. Los valores deben incluir etiquetas de idioma mediante la sintaxis `valor@idioma`.                           |
| `altLabel`      |          No | Etiquetas alternativas o sinónimos. Se pueden incluir varios valores separados con `\|`.                                                      |
| `hiddenLabel`   |          No | Etiquetas ocultas, variantes ortográficas o términos de acceso no preferentes que no deben mostrarse como sinónimos ordinarios.               |
| `notation`      |          No | Código de clasificación, notación o referencia interna asociada al recurso.                                                                   |
| `definition`    |          No | Definición del recurso. Los valores deben usar la sintaxis `valor@idioma`.                                                                    |
| `scopeNote`     |          No | Nota de alcance que explica cómo debe utilizarse el recurso. Los valores deben usar la sintaxis `valor@idioma`.                               |
| `broader`       |          No | Código de uno o varios conceptos superiores. Si hay varios, se separan con `\|`.                                                              |
| `related`       |          No | Código de uno o varios conceptos asociados. Si hay varios, se separan con `\|`.                                                               |
| `memberOf`      |          No | Código de una o varias colecciones a las que pertenece el recurso. Si hay varias, se separan con `\|`.                                        |
| `sameAs`        |          No | URI, IRI o URN externa que identifica un recurso equivalente. Se pueden incluir varios valores separados con `\|`.                            |

### Tipos de recurso

La columna `resource_type` acepta tres valores:

```text
concept
collection
conceptscheme
```

Un `concept` representa un término, categoría o unidad conceptual del vocabulario. Normalmente, la mayoría de las filas de un vocabulario serán conceptos.

Una `collection` representa una agrupación de recursos. Las colecciones son útiles cuando se necesita agrupar un conjunto de conceptos sin crear una relación jerárquica de tipo superior/inferior.

Un `conceptscheme` representa el vocabulario en su conjunto. Un vocabulario puede incluir una fila de este tipo para proporcionar etiquetas, definiciones o notas del esquema completo.

### Valores multilingües

VocLab representa etiquetas y notas multilingües mediante la siguiente sintaxis:

```text
valor@idioma
```

Por ejemplo:

```text
Climate change@en
```

Un campo puede contener varios valores multilingües separados mediante el carácter de barra vertical:

```text
Climate change@en|Cambio climático@es
```

El código de idioma debe coincidir con uno de los idiomas configurados para el vocabulario. Por ejemplo, si los idiomas disponibles del vocabulario son:

```text
en,es
```

entonces valores como los siguientes son válidos:

```text
Action@en|Acción@es
```

pero un valor como el siguiente sería inválido si `fr` no forma parte de los idiomas disponibles:

```text
Action@en|Acción@es|Action culturelle@fr
```

Las siguientes columnas usan valores multilingües:

```text
prefLabel
altLabel
hiddenLabel
definition
scopeNote
```

### Relaciones entre recursos

Las relaciones se expresan escribiendo el `code` de otro recurso.

Por ejemplo, si el vocabulario contiene este concepto:

```csv
concept,action,Action@en|Acción@es,,,,,,,,,
```

otro concepto puede referirse a él como concepto superior:

```csv
concept,platform-games,Platform games@en|Juegos de plataformas@es,,,,,,action,,,
```

Cuando una relación tiene varios destinos, se separan con `|`:

```csv
concept,metroidvania,Metroidvania@en|Metroidvania@es,,,,,,platform-games,open-world|action-rpg,,
```

VocLab valida que los códigos referenciados existan en el vocabulario.

Las principales columnas de relación son:

| Columna    | Significado                              |
| ---------- | ---------------------------------------- |
| `broader`  | Concepto superior jerárquico.            |
| `related`  | Relación asociativa con otro concepto.   |
| `memberOf` | Colección a la que pertenece el recurso. |
| `sameAs`   | URI, IRI o URN externa equivalente.      |

### Ejemplo

El siguiente ejemplo simplificado muestra un pequeño fragmento de vocabulario bilingüe:

```csv
resource_type,code,prefLabel,altLabel,hiddenLabel,notation,definition,scopeNote,broader,related,memberOf,sameAs
conceptscheme,video-game-genres,Video game genres@en|Géneros de videojuegos@es,,,,A vocabulary for describing video game genres.@en|Vocabulario para describir géneros de videojuegos.@es,,,,,
concept,action,Action@en|Acción@es,Action game@en|Videojuego de acción@es,,GEN-ACT,A genre focused on real-time challenges and player reaction.@en|Género centrado en desafíos en tiempo real y reacción del jugador.@es,,,,,
concept,platform-games,Platform games@en|Juegos de plataformas@es,Platformers@en,,GEN-PLA,A genre based on jumping movement and spatial progression.@en|Género basado en saltos movimiento y progresión espacial.@es,,action,,,
concept,metroidvania,Metroidvania@en|Metroidvania@es,,,GEN-MET,A subgenre based on interconnected exploration and progressive unlocking of abilities.@en|Subgénero basado en exploración interconectada y desbloqueo progresivo de habilidades.@es,,platform-games,action-rpg,,
collection,action-subgenres,Action subgenres@en|Subgéneros de acción@es,,,,,,,,
concept,action-rpg,Action RPG@en|RPG de acción@es,,,GEN-ARPG,A role-playing game with real-time action mechanics.@en|Juego de rol con mecánicas de acción en tiempo real.@es,,action,,action-subgenres,
```

En este ejemplo:

* `video-game-genres` es el esquema de conceptos.
* `action`, `platform-games`, `metroidvania` y `action-rpg` son conceptos.
* `platform-games` tiene `action` como concepto superior.
* `metroidvania` tiene `platform-games` como concepto superior.
* `metroidvania` está relacionado con `action-rpg`.
* `action-rpg` pertenece a la colección `action-subgenres`.

### Reglas de validación

Cuando se carga o edita un vocabulario, VocLab comprueba que:

* las columnas obligatorias existen y aparecen en el orden esperado;
* cada recurso tiene un `resource_type` válido;
* cada recurso tiene un `code` único;
* cada recurso tiene al menos un `prefLabel` válido;
* los valores multilingües usan etiquetas de idioma válidas;
* las etiquetas de idioma coinciden con los idiomas disponibles configurados para el vocabulario;
* los destinos de las relaciones en `broader`, `related` y `memberOf` existen;
* las relaciones jerárquicas no generan ciclos;
* los conceptos relacionados no están situados en la misma ruta jerárquica directa;
* los valores de `sameAs` son URI, IRI o URN válidas.

---

## Validación de vocabularios

Cuando se carga o edita un vocabulario, VocLab valida su contenido antes de almacenarlo o actualizarlo.

El proceso de validación comprueba, entre otros aspectos:

* Estructura de columnas esperada.
* Valores obligatorios.
* Tipos de recurso válidos.
* Códigos de recurso duplicados.
* Valores con etiquetas de idioma válidas.
* Uso de los idiomas disponibles del vocabulario.
* Referencias jerárquicas y asociativas válidas.
* Referencias válidas a miembros de colecciones.
* Ciclos jerárquicos.
* Conceptos relacionados situados en la misma ruta jerárquica directa.
* Valores URI, IRI o URN válidos.

Si la validación falla, VocLab informa de los errores detectados para que el usuario pueda corregir el fichero fuente o la tabla editable.

---

## Edición de vocabularios

Los vocabularios existentes pueden ser editados por usuarios autorizados.

La página de edición permite:

* Actualizar los metadatos del vocabulario.
* Cambiar la visibilidad pública o privada.
* Sustituir el fichero fuente CSV o TSV.
* Editar directamente la tabla del vocabulario.
* Revalidar el vocabulario después de los cambios.

El editor en tabla ofrece una interfaz similar a una hoja de cálculo para modificar filas y columnas sin salir de la aplicación.

![Editor de tabla de vocabulario](assets/img/image-4.png)

*Figura 4. Editor de vocabulario con interfaz similar a una hoja de cálculo.*

---

## Uso de vocabularios

La página de detalle del vocabulario ofrece varias formas de explorar sus contenidos.

Los usuarios pueden navegar por los recursos mediante:

* Vista alfabética.
* Vista jerárquica.
* Vista por grupos o colecciones.
* Caja de búsqueda.
* Panel de detalle del recurso.
* Enlaces internos entre recursos relacionados.

El panel de detalle muestra etiquetas, notas, relaciones semánticas, pertenencia a colecciones, mapeos y referencias URI externas.

![Navegador de vocabulario](assets/img/image-5.png)

*Figura 5. Navegador interactivo de vocabularios.*

---

## Visualización gráfica

VocLab incluye una visualización gráfica interactiva basada en D3.js.

La vista de grafo permite:

* Explorar visualmente las jerarquías de conceptos.
* Mostrar relaciones jerárquicas.
* Mostrar enlaces asociativos entre conceptos relacionados.
* Expandir o contraer ramas conceptuales.
* Arrastrar nodos.
* Hacer zoom y desplazarse por el grafo.
* Abrir un recurso seleccionado en el navegador del vocabulario.
* Exportar el grafo actual como SVG.

![Grafo del vocabulario](assets/img/image-6.png)

*Figura 6. Visualización gráfica interactiva del vocabulario.*

---

## Estadísticas del vocabulario

VocLab puede calcular estadísticas descriptivas de un vocabulario.

Las estadísticas incluyen:

* Número de conceptos.
* Número de colecciones.
* Número de esquemas de conceptos.
* Número de etiquetas.
* Número de definiciones y notas de alcance.
* Número de relaciones semánticas.
* Número de relaciones de mapeo.
* Número de conceptos raíz, intermedios, finales y huérfanos.
* Número de conceptos ambiguos.
* Indicadores de profundidad jerárquica.
* Indicadores de tamaño de rama.

Las estadísticas globales y las estadísticas por concepto pueden descargarse como ficheros CSV.

![Estadísticas del vocabulario](assets/img/image-7.png)

*Figura 7. Modal de estadísticas del vocabulario.*

---

## Opciones de exportación

Los vocabularios válidos pueden exportarse en varios formatos:

```text
CSV
TBL
Turtle
RDF/XML
```

Las exportaciones CSV y TBL se generan a partir de la tabla fuente almacenada.

Las exportaciones Turtle y RDF/XML se generan a partir del modelo interno del vocabulario e incluyen sentencias RDF compatibles con SKOS para:

* Esquemas de conceptos.
* Conceptos.
* Colecciones.
* Etiquetas preferentes.
* Etiquetas alternativas.
* Etiquetas ocultas.
* Definiciones.
* Notas de alcance.
* Relaciones jerárquicas.
* Relaciones asociativas.
* Miembros de colecciones.
* Conceptos superiores.
* Relaciones de mapeo.
* Enlaces `owl:sameAs`.

![Opciones de exportación](assets/img/image-8.png)

*Figura 8. Opciones de exportación tabular y RDF.*

---

## Vocabularios públicos

Un vocabulario puede hacerse público cuando ha superado la validación.

Los vocabularios públicos son visibles para visitantes anónimos desde el catálogo público. Los usuarios anónimos pueden navegar y exportar vocabularios públicos, pero no pueden editarlos ni eliminarlos.

Los vocabularios privados solo son visibles para administradores y usuarios asignados.

---

## Ficheros de ejemplo

El repositorio puede incluir ficheros CSV de ejemplo para probar la aplicación.

Ubicación sugerida:

```text
examples/
```

Posibles ejemplos:

```text
voclab_sample.csv
voclab_sample_errors.csv
voclab_ipct-mediatopic.csv
```

Estos ficheros ilustran la plantilla esperada y pueden utilizarse para probar la validación, la detección de errores y la visualización de vocabularios.

---

## Notas de seguridad

Antes de desplegar VocLab en producción:

* Sustituye las credenciales de ejemplo de `config/config.php`.
* Usa una contraseña fuerte para la cuenta del administrador.
* Limita los permisos del usuario de base de datos a la base de datos necesaria.
* Utilizar la aplicación mediante HTTPS.
* Manenter actualizados PHP, MySQL/MariaDB y el servidor web.
* Evitar subir al servidor público ficheros de desarrollo, borradores o copias de seguridad de la base de datos.

---

## Licencia

VocLab es software libre publicado bajo la **GNU General Public License v3.0**.

Puedes redistribuirlo y/o modificarlo bajo los términos de la GNU General Public License publicada por la Free Software Foundation, ya sea la versión 3 de la Licencia o cualquier versión posterior.

Consulta el fichero `LICENSE` para más detalles.
