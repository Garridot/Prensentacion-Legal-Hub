## Prueba de conocimiento con Django Rest Framework 

[Objetivo]<br>
Analizar el nivel de conocimiento de los postulantes a desarrollador de backend [Nombre de la empresa].<br> 
[Prueba lógica]<br>
Crear una API REST utilizando DJANGO REST FRAMEWORK, que brinde la siguiente funcionalidad básica y acotada de un Ecommerce.<br> 
El sistema debe tener los modelos Product, Order, OrderDetail con los siguientes atributos: 


#### Product: 
- id PK [string] 
- name [string] 
- price [float] 
- stock[int] 
#### Order:
- id PK
- date_time [datetime]

#### OrderDetail: 
- order [Order] 
- quantity [int] 
- product [Product] 


La misma debe proporcionar los siguientes end points: 

- Registrar/Editar un producto 
- Eliminar un producto 
- Consultar un producto 
- Listar todos los productos
- Modificar stock de un producto 
- Registrar/Editar una orden (inclusive sus detalles). Debe actualizar el stock del producto 
- Eliminar una orden. Restaura stock del producto 
- Consultar una orden y sus detalles 
- Listar todas las ordenes

La clase Order debe exponer un método get_total el cual calcula el total de la factura y retornar ese valor en el serializer correspondiente. También debe exponer el método get_total_usd, utilizando el API de https://www.dolarsi.com/api/api.php?type=valoresprincipales, con “dólar blue” para que te tire el precio en dolares. 

Al crear o editar una orden validar q haya suficiente stock del producto, en caso no contar con stock se debe retornar un error de validación. 

Para la implementación de la API se debe utilizar ModelViewSet, ModelSerializer. El código fuente de la api debe ser subido a un repositorio público, el cual debe ser proporcionado para su correcta examinación. A la hora de crear una orden se debe validar: 

- que la cantidad de cada producto sea mayor a 0 
- que no se repitan productos en el mismo pedido
- Implementar autenticación basada en tokens (JWT) 
- Deployar la api en producción, por ejemplo en heroku o https://www.pythonanywhere.com/, 
- Implementar test unitario para validar los endpoints.


## Ruins of Versailles | Ecommerce Project

### Arquitectura del Proyecto:

### Database del Proyecto:

![](project_images/database_project1.png)

### Serializers:
```

class OrderDetailSerializer(serializers.ModelSerializer):

    quantity = serializers.IntegerField(validators=[required,is_lower])

    class Meta:
        model  = OrderDetail       
        fields =  ('__all__')

    def validate(self, attrs):
        order    = attrs['order']
        product  = attrs['product']
        quantity = attrs['quantity'] 
        
        if OrderDetail.objects.filter(order=order,product=product).exists():
            data   = {'Error':'The product has already been requested in this order.'}
            raise serializers.ValidationError(data)

        get_prod  = Product.objects.get(name=product)     

        if get_prod.status == 'out-of-stock': 
            data   = {'Error':'This product is out-of-stock.'}
            raise serializers.ValidationError(data)     

        if get_prod.stock < quantity: 
            data   = {'Error':f'There is not enough stock left to add this product to the order. Stock available: {get_prod.stock}'}
            raise serializers.ValidationError(data) 

        return super().validate(attrs)

 
```
### ViewSets:
```
class OrderDetailView(ModelViewSet):

    serializer_class   = OrderDetailSerializer
    queryset           = OrderDetail.objects.all()

    def list(self, request):

        queryset   = OrderDetail.objects.all()
        serializer = OrderDetailSerializer(queryset, many=True)
        return Response(serializer.data)

    def create(self, validated_data):
        data = validated_data.data     

        serializer = self.get_serializer(data=data)

        if serializer.is_valid(raise_exception=True):
            action = OrderDetailActions(data['product'],data['order'],data['quantity']) 
            action.get_stock()  
            self.perform_create(serializer)

            # get total order 
            get_order   = Order.objects.get(id=data['order'])
            get_product = Product.objects.get(id=data['product'])

            get_order.total += (get_product.price * int(data['quantity']))
            get_order.save()

            # get total USD
            dolar  = get_exchange_rate()
            get_order.total_USD =  get_order.total / dolar

            get_order.save()
    
            return Response(serializer.data, status=status.HTTP_201_CREATED, ) 

    def destroy(self, request, *args, **kwargs):
        data = self.get_object()

        action = OrderDetailActions(data.product.id,data.order,data.cuantity) 
        action.return_stock()

        return super().destroy(request, *args, **kwargs)    
            
```


### Resultado Final:

![](https://github.com/Garridot/Ruins-of-Versailles_Ecommerce-Project/blob/main/project_images/image__1.png)
![](https://github.com/Garridot/Ruins-of-Versailles_Ecommerce-Project/blob/main/project_images/image__2.png)
![](https://github.com/Garridot/Ruins-of-Versailles_Ecommerce-Project/blob/main/project_images/image__3.png)
![](project_images/project1__5.png)



### 💻 Tecnologias Usadas:

![Python](https://img.shields.io/badge/python-3670A0?style=for-the-badge&logo=python&logoColor=ffdd54) ![Django](https://img.shields.io/badge/django-%23092E20.svg?style=for-the-badge&logo=django&logoColor=white) ![DjangoREST](https://img.shields.io/badge/DJANGO-REST-ff1709?style=for-the-badge&logo=django&logoColor=white&color=ff1709&labelColor=gray) 
![JavaScript](https://img.shields.io/badge/javascript-%23323330.svg?style=for-the-badge&logo=javascript&logoColor=%23F7DF1E) ![HTML5](https://img.shields.io/badge/html5-%23E34F26.svg?style=for-the-badge&logo=html5&logoColor=white) ![CSS3](https://img.shields.io/badge/css3-%231572B6.svg?style=for-the-badge&logo=css3&logoColor=white) ![Bootstrap](https://img.shields.io/badge/bootstrap-%23563D7C.svg?style=for-the-badge&logo=bootstrap&logoColor=white)
<br>
<br>
<br>

## Football Players Stats Api

El Football Players Stats API es un proyecto de Django Rest Framework diseñado para gestionar la información y las estadísticas de los jugadores de fútbol.


### Arquitectura del Proyecto:

![](project_images/diagram_project2.jpg)

## API Endpoints
- ### Jugadores:
    - **GET /players/**: Get a list of players.
    - **GET /players/{id}/**: Get the player.
    - **POST /players/**: Create a new player. (Authentication required).
    - **PUT /players/{id}/**: Update a player. (Authentication required).
    - **DELETE /players/{id}**/: Delete a player. (Authentication required).

- ### Estadísticas de los Jugadores:
    - **GET /player_stats/**: Get a list of player statistics.
    - **GET /player_stats/{id}/**: Get the player statistics.
    - **POST /api/player_stats/**: Create new player statistics. (Authentication required).
    - **PUT /player_stats/{id}/**: Update player statistics. (Authentication required).
    - **DELETE /player_stats/{id}/**: Delete player statistics. (Authentication required).

- ### Estadísticas del jugador por posición  
    - **GET /player_stats_by_position/**: Get a list of player statistics by position.  

- ### Autenticación:
    - **POST /authentication/**: Get token authentication. 
    - **POST /register/**: Register a new user. 


### Función para obtener su token de autenticación

```
data = { "email": YOUR_EMAIL, "password": YOUR_PASSWORD}
```
``` python
def get_auth_token(data):     
    data_json = json.dumps(data) # convert data to JSON format    
    headers = {"Content-Type": "application/json"} # set headers to indicate JSON content    
    response = requests.post(API_AUTH_URL, data=data_json, headers=headers) # make a POST request to the authentication API
    
    if response.status_code == 200:  # check the response status code        
        token = response.json()['access'] # Extract and return the access token from the response
        return token
    else:        
        raise Exception(f"Error during request to stats API. Status code: {response.status_code}")        
```

### Web Scraping

Esta API utiliza un proceso automatizado llamado raspado web para recopilar datos. A continuación, almacena y actualiza esta información con regularidad para asegurarse de que sigue siendo precisa y actualizada.

Este proyecto de raspado web de Python se centra en extraer las estadísticas de los jugadores de fútbol del sitio web de Transfermarkt. El script principal, main.py, utiliza la biblioteca BeautifulSoup para analizar el contenido HTML de las páginas web y extraer la información necesaria. La programación para ejecutar este script se configura mediante GitHub Actions.

### Resultado Final:

![](project_images/project2__1.png)
![](project_images/project2__2.png)
![](project_images/project2__3.png)
![](project_images/project2__4.png)

