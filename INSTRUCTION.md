# Validation instructions

## 1. Create the cluster and deploy

```bash
kind create cluster --config cluster.yml
bash bootstrap.sh
```

## 2. Check MySQL StatefulSet

```bash
kubectl get statefulset -n mysql
kubectl get pods -n mysql
kubectl get pvc -n mysql
kubectl get svc mysql -n mysql
```

Expected:
- StatefulSet `mysql` with 3/3 ready replicas
- Pods `mysql-0`, `mysql-1`, `mysql-2` in `Running` state
- PVCs `data-mysql-0`, `data-mysql-1`, `data-mysql-2` in `Bound` state
- Service `mysql` with `CLUSTER-IP` = `None` (headless)

Check probes, resources and mounted init script:

```bash
kubectl describe pod mysql-0 -n mysql
kubectl exec -n mysql mysql-0 -- ls /docker-entrypoint-initdb.d
```

## 3. Check the application

```bash
kubectl get pods -n todoapp
kubectl get deploy todoapp -n todoapp -o jsonpath='{.spec.template.spec.containers[0].env}'
```

Expected: pods in `Running` state, DB variables are read from `app-secret`.

## 4. Check that the app uses mysql-0

```bash
kubectl exec -n mysql mysql-0 -- mysql -uapp_user -papp_pass -e "SHOW TABLES FROM app_db;"
```

Expected: Django tables (`auth_user`, `django_migrations`, `lists_todo`, ...).

## 5. Open the app

```bash
kubectl port-forward -n todoapp deploy/todoapp 8080:8080
```

Open http://localhost:8080, create a user and a todo list. Then verify the data is stored in MySQL:

```bash
kubectl exec -n mysql mysql-0 -- mysql -uapp_user -papp_pass -e "SELECT * FROM app_db.lists_todolist;"
```