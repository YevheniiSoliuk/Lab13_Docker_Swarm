# Lab13_Docker_Swarm
## 1. Zawartość pliku docker-stack.yml

```yaml
version: '3'

services:
  apache:
    image: apache-httpd
    ports: 
      - "8080:80"
    volumes:
      - ./public_html/:/var/www/html/
    depends_on:
      - php
      - mysql
    restart: unless-stopped
    networks:
      - frontend
      - backend
    deploy:
      replicas: 1
      update_config:
        parallelism: 2
        delay: 10s
      restart_policy:
        condition: on-failure

  php:
    image: php:7.2.7-fpm-alpine3.7
    volumes:
      - ./public_html/:/var/www/html/
    networks:
      - backend
    deploy:
      replicas: 2
      update_config:
        parallelism: 1
        delay: 10s
      restart_policy:
        condition: on-failure
  
  mysql:
    image: mysql:latest
    environment:
      MYSQL_DATABASE: docker_database
      MYSQL_ROOT_PASSWORD: rootpass
    networks:
      - frontend
    deploy:
      replicas: 1
      update_config:
        parallelism: 2
        delay: 10s
      placement:
        constraints: [node.role == manager]
      restart_policy:
        condition: on-failure
  
  phpmyadmin:
    image: phpmyadmin/phpmyadmin
    ports:
      - '8000:80'
    restart: always
    environment:
      PMA_HOST: mysql
      MYSQL_ROOT_PASSWORD: rootpass
    links:
      - mysql
    depends_on:
      - mysql
    networks:
      - frontend
    deploy:
      replicas: 1
      update_config:
        parallelism: 2
        delay: 10s
      restart_policy:
        condition: on-failure

networks:
  frontend:
  backend:
```

W danym pliku w odróżnieniu do *docker-compose.yml* są pare rzeczy, mianowicie zamiast używania poleceń do budowania obrazu używa się wcześnie zbudowanych obrazów oraz dodana sekcja **deploy.**

## 2. Uruchomienia usługi w klastrze

Inicjalizacja węzła managera poleceniem 
```docker swarm init```

![Zrzut ekranu 2022-06-10 154302](https://user-images.githubusercontent.com/78736395/173085664-98ecf38e-aa4c-4594-b821-e3e650873cd4.jpg)


Sprawdzenie czy węzeł został poprawnie zaincjalizowany poleceniem
```docker node ls```

![Zrzut ekranu 2022-06-10 155055](https://user-images.githubusercontent.com/78736395/173085732-be937b23-dc73-4968-bb74-7a7c4df70671.jpg)


Wynik uruchomienie klastra poleceniem 
```docker stack deploy --compose-file docker_stack.yml swarm```

![Zrzut ekranu 2022-06-10 154611](https://user-images.githubusercontent.com/78736395/173085773-de5b0c21-c7b0-4974-906c-0703b803eaf0.jpg)


Sprawdzenie czy wszystkie serwisy wraz z ich kopiami zostały uruchomione poleceniem
```docker service ls```

![Zrzut ekranu 2022-06-10 154803](https://user-images.githubusercontent.com/78736395/173085866-24a1c5ff-ee6a-440c-bd64-e56fe4fbe621.jpg)


Sprawdzenie uruchomienia wszystkich kontenerów w stack-u poleceniem
```docker stack ps swarm```

![Zrzut ekranu 2022-06-10 155415](https://user-images.githubusercontent.com/78736395/173085931-a49fb4cc-5164-4389-88eb-0dd698c8a3c2.jpg)


## 3. Przedstawienie dostępu do stron

Uruchomienie strony internetowej

![Zrzut ekranu 2022-06-10 155702](https://user-images.githubusercontent.com/78736395/173085978-e6fbffc3-9d9e-4362-bf3f-0c6d72d514a3.jpg)


Uruchomienie panelu phpmyadmin

![Zrzut ekranu 2022-06-10 155812](https://user-images.githubusercontent.com/78736395/173086020-15cad61c-7c6a-4e21-b178-024e95e7bbab.jpg)


## 4. Uzasadnienie wyboru liczby replik

Wybrałem dla serwisu php dwie repliki, ponieważ php przesyła swoje pliki do volumenu z którego korzysta serwer apacha oraz może korzystać z bazy danych, zatem obrabia dużo danych i musi mieć replikę, która będzie mogła zamienić oryginał w sytuacji awaryjnej. Natomiast serwis bazy danych ma jędną replikę, bo inaczej one będą kolidowały się między sobą.
