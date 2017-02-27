<?php
/*
 * This file is part of FacturaSctipts
 * Copyright (C) 2016  Carlos Garcia Gomez  neorazorx@gmail.com
 * Copyright (C) 2016  Luis Miguel Pérez Romero luismipr@gmail.com
 *
 * This program is free software: you can redistribute it and/or modify
 * it under the terms of the GNU Lesser General Public License as
 * published by the Free Software Foundation, either version 3 of the
 * License, or (at your option) any later version.
 *
 * This program is distributed in the hope that it will be useful,
 * but WITHOUT ANY WARRANTY; without even the implied warranty of
 * MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
 * GNU Lesser General Public License for more details.
 * 
 * You should have received a copy of the GNU Lesser General Public License
 * along with this program.  If not, see <http://www.gnu.org/licenses/>.
 */
 
 
/*
 * @author itacaswl
 * @copyright 2016, itacaswl. All Rights Reserved.
 */

require_model('factura_cliente.php');
require_model('grupo_clientes.php');
require_model('cliente.php');
require_model('asiento_factura.php');

/**
 * Description of facturargrupo
 *
 * @author itacaswl
 */
class facturargrupo extends fs_controller
{

   public $documento;
   public $gclientes;
   public $clientes;
   public $codgrupo;
   public $clicente;
   

   public function __construct()
   {
      parent::__construct(__CLASS__, 'Facturargrupo', 'ventas', FALSE, FALSE);      
   }
   
   protected function private_core()
   {
   	
   	$this->share_extensions();
      $this->documento = FALSE;
      $this->cliente = FALSE; 
      
      if (isset($_REQUEST['id']))
       {	
   	
         $factura = new factura_cliente();
         $this->documento = $factura->get($_REQUEST['id']);
         
         
         $this->gclientes = $this->db->select("select * from gruposclientes;");
         
         if (isset($_REQUEST['codgrupo']))
          {	
           $this->codgrupo=$_REQUEST['codgrupo'];
            $this->clientes = $this->db->select("select * from clientes where codcliente NOT LIKE '".$this->documento->codcliente."' AND codgrupo LIKE '".$_REQUEST['codgrupo']."' AND NOT debaja;");
            
            
             if (isset($_REQUEST['generar']))
             {	        
                  // $this->template = FALSE;
                  //echo var_dump($this->clientes);
                foreach ($this->clientes as $clienteaux) {
                	
                	//echo $clienteaux["codcliente"].'<br/>';
                
                  $this->generar_factura($clienteaux["codcliente"]);
              }  

            
             }
            
            
            
            
           
          }      
         

        }



   }
   
   /**
    * Añadimos las extensiones (los botones).
    */
   private function share_extensions() {
      $fsext = new fs_extension();
      $fsext->name = 'Facturargrupo';
      $fsext->from = __CLASS__;
      $fsext->to = 'ventas_factura';
      $fsext->type = 'button';
      $fsext->text = '<span class="glyphicon glyphicon-random" aria-hidden="true"></span><span>&nbsp; Facturar a Grupo</span>';
      $fsext->save();


   }




   public function url()
   {
      if ($this->documento)
      {
      	if ($this->codgrupo)
           return 'index.php?page=' . __CLASS__ . '&id=' . $_REQUEST['id'] . '&codgrupo='. $_REQUEST['codgrupo']. '';
         else 
           return 'index.php?page=' . __CLASS__ . '&id=' . $_REQUEST['id'] ;
      }
      else
      {
         return 'index.php?page=' . __CLASS__;
      }
   }

 

   private function generar_factura($codcliente) 
   {
   
            //En $this->documento ya tenemos la factura original

            if ($this->documento)
            {

                  /**
                   * Si nos llega la variable copiar es que han pulsado el botón
                   * de copiar, así que copiamos la factura.
                   */
                  $factura = clone $this->documento;
                  $factura->idpresupuesto = NULL;
                  $factura->idpedido = NULL;
                  $factura->idalbaran = NULL;
                  $factura->idfactura = NULL;
                  $factura->idasiento = NULL;
                  $factura->idasientop = NULL;
                  $factura->fecha = Date('d-m-Y');
                  $factura->hora = date('H:i:s');
                 // $factura->observaciones = $_REQUEST['observaciones'];
                 // $factura->codserie = $this->serie->codserie;
                 // $factura->codalmacen = $this->almacen->codalmacen;
                 // $factura->codagente = $this->user->codagente;
                 
                 
                 $cli0 = new cliente();
                 $this->cliente = $cli0->get($codcliente);

                  //cliente:
                  if ($this->cliente)
                  {
                     foreach ($this->cliente->get_direcciones() as $d)
                     {
                        if ($d->domfacturacion)
                        {
                           $factura->codcliente = $this->cliente->codcliente;
                           $factura->cifnif = $this->cliente->cifnif;
                           $factura->nombrecliente = $this->cliente->razonsocial;
                           $factura->apartado = $d->apartado;
                           $factura->ciudad = $d->ciudad;
                           $factura->coddir = $d->id;
                           $factura->codpais = $d->codpais;
                           $factura->codpostal = $d->codpostal;
                           $factura->direccion = $d->direccion;
                           $factura->provincia = $d->provincia;
                           break;
                        }
                     }

                     if ($factura->save())
                     {

                        /// también copiamos las líneas de la factura
                        foreach ($this->documento->get_lineas() as $linea)
                        {
                           $newl = clone $linea;
                           $newl->idlinea = NULL;
                           $newl->idfactura = $factura->idfactura;
                           $newl->save();
                        }
                        $this->generar_asiento($factura);
                        $this->new_message('<a href="' . $factura->url() . '">Documento '.$factura->codigo.'</a> de factura generado correctamente.');
                     }
                     else
                     {
                        $this->new_error_msg('Error al generar el documento.');
                     }
                  }
                  else
                  {
                     $this->new_error_msg('No se ha encontrado el cliente');
                  }
               
            }  
   
   
   
   
   }


   /**
    * Genera el asiento para la factura, si procede
    * @param factura_cliente $factura
    */
   private function generar_asiento(&$factura)
   {
      if($this->empresa->contintegrada)
      {
         $asiento_factura = new asiento_factura();
         $asiento_factura->generar_asiento_venta($factura);
         
         foreach($asiento_factura->errors as $err)
         {
            $this->new_error_msg($err);
         }
         
         foreach($asiento_factura->messages as $msg)
         {
            $this->new_message($msg);
         }
      }
      else
      {
         /// de todas formas forzamos la generación de las líneas de iva
         $factura->get_lineas_iva();
      }
   }




}
