# README

## Description
Ce projet est une implémentation d'une API REST à l'aide d'AWS API Gateway et AWS Lambda. L'API permet de gérer une liste d'éléments (éventuellement des produits), en permettant les opérations suivantes :
- Récupération de tous les éléments (**GET**)
- Ajout d'un élément (**POST**)
- Mise à jour d'un élément existant (**PUT**)
- Suppression d'un élément (**DELETE**)

L'API est construite de manière serverless et fonctionne sans état.

---

## Fonctionnalités principales
- **GET** `/items` : Retourne la liste de tous les éléments.
- **POST** `/items` : Ajoute un nouvel élément à la liste.
- **PUT** `/items` : Met à jour les informations d'un élément existant.
- **DELETE** `/items` : Supprime un élément de la liste.

---

## Configuration et déploiement

### **1. Création de la fonction Lambda**
- La fonction Lambda a été créée à l'aide de Python.
- Voici le code principal utilisé pour la gestion des requêtes :

```python
import json

data = {
    "items": [
        {"id": 1, "name": "Item 1", "price": 10.99},
        {"id": 2, "name": "Item 2", "price": 15.99},
        {"id": 3, "name": "Item 3", "price": 20.99},
    ]
}

def lambda_handler(event, context):
    http_method = event.get("httpMethod", "")

    if http_method == "GET":
        return build_response(200, data)

    elif http_method == "POST":
        body = json.loads(event.get("body", "{}"))
        if not body.get("id") or not body.get("name") or not body.get("price"):
            return build_response(400, {"error": "Missing fields: id, name, price required"})
        data["items"].append(body)
        return build_response(201, {"message": "Item added", "data": data})

    elif http_method == "PUT":
        body = json.loads(event.get("body", "{}"))
        item_id = body.get("id")
        if not item_id:
            return build_response(400, {"error": "Missing field: id required"})
        for item in data["items"]:
            if item["id"] == item_id:
                item.update(body)
                return build_response(200, {"message": "Item updated", "data": data})
        return build_response(404, {"error": "Item not found"})

    elif http_method == "DELETE":
        body = json.loads(event.get("body", "{}"))
        item_id = body.get("id")
        if not item_id:
            return build_response(400, {"error": "Missing field: id required"})
        for i, item in enumerate(data["items"]):
            if item["id"] == item_id:
                del data["items"][i]
                return build_response(200, {"message": "Item deleted", "data": data})
        return build_response(404, {"error": "Item not found"})

    else:
        return build_response(405, {"error": "Method not allowed"})

def build_response(status_code, body):
    return {
        "statusCode": status_code,
        "headers": {
            "Content-Type": "application/json",
            "Access-Control-Allow-Origin": "*",
            "Access-Control-Allow-Methods": "GET, POST, PUT, DELETE",
        },
        "body": json.dumps(body)
    }
```

---

### **2. Configuration d'API Gateway**
- Une API REST a été créée avec AWS API Gateway.
- Les ressources suivantes ont été définies :
  - Ressource : `/items`
  - Méthodes : `GET`, `POST`, `PUT`, `DELETE`
- Chaque méthode est intégrée à la fonction Lambda via l'option **"Intégration de proxy Lambda"**.

### **3. Tests avec Postman**
L'API a été testée avec Postman en utilisant les URL déployées dans API Gateway.

#### **Exemple de requêtes**
- **GET** `/items`
  - Retourne tous les éléments de la liste.

- **POST** `/items`
  - Exemple de body :
    ```json
    {
        "id": 4,
        "name": "New Item",
        "price": 30.99
    }
    ```

- **PUT** `/items`
  - Exemple de body :
    ```json
    {
        "id": 2,
        "name": "Updated Item",
        "price": 25.99
    }
    ```

- **DELETE** `/items`
  - Exemple de body :
    ```json
    {
        "id": 3
    }
    ```

---

## **Déploiement sur AWS**
- L'API a été déployée sur le stage `dev`.
- L'URL de l'API est :
  ```
  https://ka7cj34obb.execute-api.eu-west-3.amazonaws.com/dev/
  ```

