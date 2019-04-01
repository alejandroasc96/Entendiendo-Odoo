# Acción Planificada en Odoo 11
En Odoo, podemos ejecutar tareas planificadas que se ejecuten cada cierto tiempo. Por ejemplo, podemos enviar un email cada mes con el resumen de las ventas.

Para este ejemplo vamos a automatizar la comprobación de las suscripciones de los servicios que tienen actualmente contratados nuestro clientes.En el caso que solo le queden 30 días para que se le caduque el servicio se le mandará un e-mail notificándole que debe renovar.

 Antes de empezar con dicha tarea hay que tener en cuenta los siguientes aspectos:
 
 - Tener cargadas las dependencias tanto de **product** como **account** dentro de nuestro `__manifest__.py`
- Tener instalados dentro de nuestro Odoo el módulo de **Gestión de ventas** y **Facturación**
- Habernos creado un módulo de Odoo donde poder empezar a trabajar.
-  Tener conocimientos de [herencia](https://github.com/alejandroasc96/Documentacion/tree/herenciaOdoo) en  Odoo
- Tener conocimientos de los [campos calculados](https://github.com/alejandroasc96/Documentacion/tree/campoCalculadoOdoo)
- Tener conocimiento de [campos relacionados](https://github.com/alejandroasc96/Documentacion/tree/campoRelacionadoOdoo)

Para la creación de nuestra acción planificada deberemos configurar los siguientes aspectos:
- Crear nuestro campo en el formulario **Producto** (producto.template)
- Calcular los días que quedan para que se finalice la suscripción.
-  Crear nuestra vista de acción planificada.
- Crear nuestra función de python.


## 1.- Creando nuestro nuevo Campo en producto
Para poder gestionar las suscripciones de nuestro producto deberemos crear un nuevo campo encargado de almacenar el número de días que va a tener nuestra suscripción. 

*__Creación del modelo__*
~~~python
class domain(models.Model):
    _name = 'product.template'
    _inherit ='product.template'

    subscription = fields.Integer(string='Días hasta caducidad')
~~~
*__Creando la vista__*
~~~python
<record id="view_caducidad_producto_form" model="ir.ui.view">
      <field name="name">product.template.caducidad.producto.form</field>
      <field name="model">product.template</field>
      <field name="inherit_id" ref="product.product_template_form_view"/>
      <field name="arch" type="xml">
      <xpath expr="//page[@name='general_information']/group/group[@name='group_standard_price']/div[@name='standard_price_uom']" position="after">
        <field name="subscription"/>
      </xpath>
      </field>
    </record>
~~~
## 2.- Calculando los días que faltan para que finalice la suscripción
~~~python
class campo_calculado(models.Model):
    _name = 'account.invoice.line'
    _inherit = 'account.invoice.line'

    campoCalculado = fields.Integer(string='Días que han pasado',compute="_campocalculado",store=False)
    # referenciamos el campo subscriptionDays, que se encuentra en el modelo product.temple a account.invoice.line con el nombre que aparece a la izquierda
    
    rel_field = fields.Integer(string='Fecha Caducidad Establcida',related='product_id.subscriptionDays')
    total = fields.Integer(string='Días restantes ',compute='_total',store=False)

    @api.one
    @api.depends('create_date')
    def _campocalculado(self):
        for r in self:
            r.campoCalculado = (datetime.now() - datetime.strptime(r.create_date, '%Y-%m-%d %H:%M:%S')).days
            

    @api.one
    @api.depends('campoCalculado', 'rel_field')
    def _total(self):
        self.total = self.rel_field - self.campoCalculado 
~~~

## 3.- Creando nuestra vista de acción planificada

~~~python
<record forcecreate="True" id="caducidad_productos_v1" model="ir.cron">
      <field name="name">Revision de Suscripciones Vencidas</field>
      <field eval="True" name="active" />
      <field name="user_id" ref="base.user_root" />
      <field name="interval_number">24</field>
      <field name="interval_type">hours</field>
      <field name="nextcall" eval="(DateTime.now() + timedelta(days=1)).strftime('%Y-%m-%d 05:00:00')" />
      <field name="numbercall">-1</field>
      <field ref="model_account_invoice_line" name="model_id" />
        <field name="state">code</field>
      <field name="code">model.caducidad_productos()</field>
      <field eval="False" name="doall"/>
      <field name="function">True</field>

    </record>
~~~
*__Explicación__*
~~~xml
<record  forcecreate="True"  id="caducidad_productos_v1"  model="ir.cron">
~~~
En esta sentencia estamos forzando la creación de nuestra vista a la cual se le da un **id** y se la está vinculando con el modelo **ir.cron** que es el modelo específicamente creado por Odoo para todas las acciones automatizadas. Este modelo contiene todas las acciones automatizadas y siempre debe especificarse.


~~~xml
<field  name="name">Revision de Suscripciones Vencidas</field>
~~~
Le damos un nombre a nuestra acción planificada.


~~~xml
<field name="user_id" ref="base.user_root" />
~~~
En esta sentencia estamos indicando el usuario que lanza la acción que por normal general será `base.user_root`
dado que de es el usuario que posee todos los servicios.

~~~xml
<field name="interval_number">24</field>
<field name="interval_type">hours</field>
~~~
En estas dos sentencias estamos indicando la periodicidad  con la cual se van a lanzar nuestras acciones, en este caso es cada 24 horas
*__Consejo:__* las opciones para 'interval_type' son 'minutes', 'hours', 'days', 'work_days', ’weeks’ y ‘months’.


~~~xml
<field  name="nextcall"  eval="(DateTime.now() + timedelta(days=1)).strftime('%Y-%m-%d 05:00:00')"  />
~~~
Aquí le estamos diciendo a odoo que la siguiente llamada para ejecutar la acción debe producirse a la 05:00.

~~~xml
<field  name="numbercall">-1</field>
~~~
El número de llamadas en nuestro caso debe ser infinito así que le damos el valor `-1` para que se lance siempre.
*__Consejo:__* en el caso de solo querer que se lance dos veces se le pondría el valor 2 y así con todas las posibilidades que queramos darle.

~~~xml
<field  ref="model_account_invoice_line"  name="model_id"  />
~~~
Se le indica que esta acción va a trabajar sobre el modelo **`account.invoice.line`**

~~~xml
<field  name="code">model.caducidad_productos()</field>
~~~
Se le está indicando que método debe ejecutar cuando le lanza la acción ( en nuestro caso es el método que creamos más adelante en el punto 3)

~~~xml
<field  eval="False"  name="doall"/>
~~~
La siguiente línea, `doall` tiene dos opciones: True o False. Cuando el campo se establece en False, le está diciendo a Odoo que las acciones automatizadas perdidas deben ejecutarse cuando el servidor se reinicia si faltaban.

## 4.- Creamos  nuestra función de python.

~~~python
class caducidad_productos(models.Model):
    _name = 'account.invoice.line'
    _inherit = 'account.invoice.line'

    @api.model
    def caducidad_productos(self):
        total_dias = self.search([])
        print ("### Revisando las Facturas Vencidas")
        for r in total_dias:
            ale = r.rel_field - ((datetime.now() - datetime.strptime(r.create_date, '%Y-%m-%d %H:%M:%S')).days)
            _logger.warning("----------------------------" + str(ale))
            if (ale<=30):
                mail_from = r.invoice_id.user_id.email
                mail_to = r.invoice_id.partner_id.email
                userID = r.invoice_id.user_id.id

                mail_vals = {
                            'subject': 'Notificacion de Facturas Vencidas',
                            'author_id': userID, #my_user.id,
                            'email_from': mail_from,
                            'email_to': mail_to,
                            'message_type':'email',
                            'body_html': 'Se te están caducadon las suscripciones',
                                }
                
                mail_id = self.env['mail.mail'].create(mail_vals)
                mail_id.send()
                _logger.warning("----------------------------" + str(r.invoice_id))
~~~


