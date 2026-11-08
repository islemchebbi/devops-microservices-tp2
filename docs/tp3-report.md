# TP3 - Réseau Docker et Docker Compose

## Tâche 1 - Réseau et DNS
- bridge (par défaut) : driver bridge, sous-réseau 172.17.0.0/16, passerelle 172.17.0.1
- tp3-net (défini par l'utilisateur) : driver bridge, sous-réseau 172.20.0.0/16, passerelle 172.20.0.1
- nslookup tp3-web -> 172.20.0.2 (serveur DNS 127.0.0.11)

## Tâche 2 - compose.yaml
Voir compose.yaml à la racine du projet. docker compose config valide 2 services et le réseau devops-microservices-tp2_app-net.

## Tâche 3 - Déploiement et tests
- 8001/health -> product-service OK
- 8001/products -> Laptop 1200, Keyboard 80, Mouse 40
- 8002/health -> order-service OK
- 8002/orders -> liste vide
- POST commande product_id=1 quantity=2 -> Laptop, total 2400.0
- DNS : gethostbyname('product-service') -> 172.20.0.2

## Tâche 4 - Panne et rétablissement
- Après docker compose stop product-service, order-service est resté Up.
- POST /orders a retourné HTTP 500 ; les logs montrent httpx.ConnectTimeout.
- Après docker compose start product-service, la commande product_id=2 a réussi (Keyboard, total 80.0).

## Questions du rapport
Q1. Bridge par défaut : pas de DNS par nom entre conteneurs. Bridge défini par l'utilisateur : DNS automatique et meilleure isolation.
Q2. Le DNS intégré de Docker résout les noms de services en adresses IP de conteneurs. Order Service utilise http://product-service:8001 car l'IP peut changer, mais pas le nom.
Q3. services définit les conteneurs, build définit comment construire les images, ports publie les ports hôte:conteneur, environment transmet la configuration d'exécution, networks rattache les services à un réseau partagé.
Q4. Les fichiers Dockerfile.reference copient des fichiers depuis la racine du projet, donc le contexte de build doit être la racine (.).
Q5. up -d construit et démarre en arrière-plan ; stop arrête les conteneurs mais les conserve ; down supprime les conteneurs et le réseau du projet (pas le code source ni les images par défaut).
Q6. depends_on contrôle uniquement l'ordre de démarrage ; il ne garantit pas que le service est prêt à répondre en HTTP.
Q7. Order Service est resté actif, mais les requêtes ont échoué avec HTTP 500 (ConnectTimeout) car Product Service était injoignable et il n'y a pas de gestion d'erreur.
Q8. DNS : docker compose exec order-service python -c "import socket; print(socket.gethostbyname('product-service'))". Réseau : docker network inspect devops-microservices-tp2_app-net.

