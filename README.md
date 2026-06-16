# Portfolio

### Build & Deploy
1. Build the app

    ```npm run build```

    Can be tried locally with

    ```node build/index.js```

2. Build image

    ```docker build -t ericryhr/portfolio:tagname .```

    Optionally run locally

    ```docker run -p 3000:3000 ericryhr/portfolio:tagname```

3. Push image

    ```docker push ericryhr/portfolio:tagname```