
```ts
import swaggerUi from 'swagger-ui-express';  
import swaggerJsdoc from 'swagger-jsdoc';  
  
export default (app) => {  
  const options = {  
    swaggerDefinition: {  
      openapi: '3.0.0',  
      info: {  
        title: 'My API',  
        version: '1.0.0',  
        description: 'My REST API'  
      },  
      servers: [  
        {  
          url: `http://localhost:${process.env.PORT}`  
        }  
      ]  
    },  
    apis: ['./*.js']  
  };  
  
  const specs = swaggerJsdoc(options);  
  
  app.use('/api-docs', swaggerUi.serve, swaggerUi.setup(specs));  
};
```

