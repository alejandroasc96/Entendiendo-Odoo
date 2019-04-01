# Campo Calculado
Los campos pueden tener valores calculados por una función, en vez de simplemente leer un valor almacenado en una base de datos. Un campo calculado es declarado como un campo regular, pero tiene el argumento “compute” adicional con el nombre de la función que se usará para calcularlo.
```python
class  campo_calculado(models.Model):
_name =  'account.invoice.line'
_inherit =  'account.invoice.line'

campoCalculado = fields.Integer(string='Días que han pasado',compute="_campocalculado",store=False)
@api.one
@api.depends('create_date')
def  _campocalculado(self):
	self.campoCalculado = self.peso + self.gravedad
```
El código anterior agrega un campo nuevo  `campoCalculado`  y el método  `_campocalculado`  que sera usado para calcular el campo. El nombre de la función es pasado como una cadena, pero también es posible pasarla como una referencia obligatoria (el identificador de la función son comillas).
*__Explicación:__*

 - **`@api.one`** este decorador hará que nuestro self solo tenga un registro.
   
 -  Si en vez de esto usamos  **`@api.multi`**, representara un conjunto
   de registros y nuestro código necesitará gestionar la iteración sobre
   cada registro.
   
  - El  **`@api.depends`**  es necesario si el calculo usa otros campos:
   le dice al servidor cuando re-calcular valores almacenados o en
   cache. Este acepta uno o mas nombres de campo como argumento y la
   notación de puntos puede ser usada para seguir las relaciones de
   campo.
-   **`@api.constrains(fld1,...)`** se utiliza para las funciones de validación para identificar en qué cambios debe activarse la comprobación de validación
-   **`@api.onchange(fld1,...)`**  se utiliza para las funciones de cambio para identificar los campos en el formulario que desencadenará la acción

En el caso de que queramos **guardar** nuestro **campo calculado** deberemos ponerle **`store=True`** .

            
