# ocp-dog-rbac

Repositorio Helm con todos los objetos RBAC necesarios para la aplicación **ocp-dog** en el proyecto de OpenShift `guscontre-dev`. Aquí se definen exclusivamente recursos declarativos (ServiceAccounts, Roles y RoleBindings); el resto de los componentes de la app viven en su propio chart.

## Componentes generados

### ServiceAccounts
| Componente | ServiceAccount | Uso principal |
|------------|----------------|---------------|
| Frontend | `sa-ocp-dog-frontend` | Servir la SPA pública y consumir la API frontal. |
| Frontend API | `sa-ocp-dog-frontend-api` | Gateway entre el frontend y el backend de datos. |
| Backend API | `sa-ocp-dog-backend-api` | Acceso a ConfigMaps/Services necesarios para operar contra la base. |
| Database | `sa-ocp-dog-database` | Servicio de PostgreSQL interno del proyecto. |
| CI/CD | `sa-ocp-dog-deployer` | Pipelines de despliegue automatizados. |

### Roles
- `ocp-dog-app-basic`: permisos de solo lectura sobre `ConfigMaps`, `Services`, `Endpoints` y `Pods` en el namespace.
- `ocp-dog-deployer-full`: permisos de creación, actualización, lectura y borrado sobre los recursos necesarios para desplegar (ConfigMaps, Secrets, Services, Pods, PVCs, Deployments, ReplicaSets, StatefulSets, DaemonSets, HPAs y Routes) y lectura de logs de pods dentro del namespace.

### RoleBindings
Cada ServiceAccount de la aplicación queda enlazada al rol `ocp-dog-app-basic`, mientras que `sa-ocp-dog-deployer` recibe el rol `ocp-dog-deployer-full`, asegurando el principio de mínimos privilegios.

## Despliegue con Helm
```bash
helm upgrade --install ocp-dog-rbac ./charts/ocp-dog-rbac -n guscontre-dev
```
El chart asume que el namespace ya existe; si estás creando un clúster nuevo puedes habilitar la creación automática con `--set createNamespace=true`.

## Automatización con GitHub Actions
Este repositorio incluye el workflow `.github/workflows/rbac-deploy.yml`. Para usarlo:
1. Define los secretos `OPENSHIFT_SERVER`, `OPENSHIFT_TOKEN` y, opcionalmente, `OPENSHIFT_NAMESPACE` (por defecto `guscontre-dev`).
2. Lanza el workflow manualmente (`workflow_dispatch`) o empuja cambios a la rama `main`.
3. El workflow instalará `kubectl`, `oc`, `helm`, hará `oc login` y ejecutará el mismo comando Helm mostrado arriba.

Con esto puedes mantener el RBAC sincronizado en cualquier clúster de OpenShift únicamente con manifestos declarativos y Helm.
