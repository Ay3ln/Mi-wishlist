# Mi wishlist

Wishlist estática publicada en GitHub Pages y conectada al proyecto Firebase `wishlist-aef0f`.

## Causa del problema

El código del editor ya guarda en `wishlistGuides/{guideId}` y los enlaces `#view=...` leen esa misma guía desde Firestore, también en tiempo real. El bloqueo era que la cuenta administradora seguía configurada con el marcador `REEMPLAZAR_CON_TU_CORREO_VERIFICADO`, por lo que el editor no la reconocía y Firestore rechazaba sus escrituras. La URL raíz sin `#view=...` tampoco identifica una guía compartida: muestra el estado inicial local.

La cuenta autorizada está configurada ahora como `ayelencononir@gmail.com` en el editor, el panel y las reglas. Firestore continúa siendo la fuente de verdad para enlaces compartidos; el editor espera confirmación de escritura y muestra errores explícitos. Las guías se pueden leer públicamente por su ID, pero solo la cuenta verificada configurada puede crearlas, editarlas o eliminarlas. No se borran ni migran documentos de wishlist.

## Configurar Firebase

1. La cuenta administradora configurada es `ayelencononir@gmail.com`. Si necesitas cambiarla, sustituye el correo por la misma dirección verificada en `index.html`, `admin.html` y `firestore.rules`.
2. En Firebase Console, proyecto `wishlist-aef0f` > **Authentication > Sign-in method**, habilita **Email/Password** y **Anonymous**. En **Users**, crea tu cuenta administradora y verifica ese correo.
3. Confirma que Firestore Database existe en el proyecto `wishlist-aef0f`. No borres ni reinicies la base.
4. Desde esta carpeta, instala Firebase CLI si hace falta, inicia sesión y despliega las reglas:

	```powershell
	npm install -g firebase-tools
	firebase login
	firebase deploy --only firestore:rules --project wishlist-aef0f
	```

El despliegue de reglas modifica permisos del proyecto Firebase, no los datos. Hazlo después de sustituir el correo. No publiques `firestore.rules` como sustituto del comando de despliegue.

El panel y el editor comprueban el correo verificado para la interfaz, pero la autorización real la aplican las reglas de Firestore. No hay contraseñas ni claves privadas en el frontend. La configuración web de Firebase es pública por diseño.

Para editar una guía, inicia sesión en `admin.html` con la cuenta verificada y usa **Editar esta wishlist**. La sesión de Firebase se conserva al abrir el editor. Comparte el enlace `#view=...` copiado desde el botón **Compartir enlace** de una guía ya guardada; la URL raíz no identifica una guía y puede mostrar solo el estado local/default de cada navegador. Los visitantes pueden consultar las guías y la disponibilidad de regalos, pero no pueden listar guías ni leer nombres/fechas privados de las reservas. Las reservas nuevas siguen guardándose en los documentos separados existentes. Los documentos antiguos no se eliminan; el panel conserva su opción explícita de migración.

## Publicar en GitHub Pages

1. Guarda los archivos modificados en VS Code.
2. Revisa `git status` para confirmar los archivos y cambios. Añade los archivos del sitio y las reglas/configuración necesarias, crea un commit y súbelo a la rama configurada en GitHub Pages (`main` en este proyecto). Por ejemplo:

	```powershell
	git add index.html admin.html README.md firebase.json firestore.rules wishlist.gif
	git commit -m "Fix shared wishlist editing and access"
	git push origin main
	```

3. En GitHub, abre **Settings > Pages** y comprueba que la fuente sea la rama `main` y la carpeta raíz. Revisa **Actions** hasta que el despliegue de Pages termine correctamente.
4. Abre `https://4y3ln.github.io/Mi-wishlist/` y `https://4y3ln.github.io/Mi-wishlist/admin.html`. Comprueba una edición real desde la cuenta autorizada.
5. Si sigues viendo contenido antiguo, haz una recarga forzada (`Ctrl+F5`), prueba una ventana privada y confirma el último commit de la rama. La subida a GitHub y el despliegue de Pages son pasos separados; este checkout no puede publicarlos automáticamente.

## Comprobar

1. Inicia sesión en `admin.html`, abre **Editar esta wishlist**, cambia un texto y guarda. Debe aparecer “Cambios guardados y sincronizados”.
2. En Firebase Console > Firestore Database, abre `wishlistGuides/{guideId}` y confirma que el campo modificado cambió. El proyecto conserva las colecciones y documentos actuales.
3. Recarga el editor y comprueba que aparece el texto de Firestore; repite desde otro navegador/dispositivo con el enlace público para verificar la versión compartida.
4. Desconecta la red o usa una cuenta no autorizada e intenta guardar. La interfaz debe comunicar el error y Firestore debe rechazar la escritura.
5. En una ventana privada, abre `admin.html` y trata de leer `wishlistGuides` como listado y `wishlistReservations/{guideId}/items`. Las reglas deben denegar ambas consultas; un visitante sí puede leer el documento de guía por su ID y los estados públicos.

La verificación real depende de configurar Authentication, desplegar `firestore.rules`, subir los cambios a GitHub y esperar Pages. Ninguno de esos despliegues se considera completado solo porque el archivo exista en el checkout local.