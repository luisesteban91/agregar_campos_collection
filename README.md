    use Illuminate\Support\Arr;
    
    public function index(Request $request){
        $solicitudes_actuales = array();

        #OBTENER EL NUMERO DEL COMITE ACTUAL
        $comite_actual = Comite::select('id')->where('fecha_fin', null)->get();

        foreach($comite_actual as $comite){
            $id_comite = $comite->id;
        }

        // #OBTENER TODAS LAS SOLICITUDES DEL COMITE
        $solicitudes_comite = Solicitud::select('id_solicitud')->where('comite_id', $id_comite)->get();
        
        #convertir un objeto stdClass a array
        $solicitudes_comite = json_decode(json_encode($solicitudes_comite),true);
        
        #CREAR UN ARRAY CON LOS PUEROS VALORES QUITANDO LA CLAVE AL ARRAY (CONVERTIR MATRIZ EN ARRAY)
        $solicitudes_comite = array_column($solicitudes_comite, 'id_solicitud');

        #REALIZAR LA PETICION TIPO GET A SERVICIOS-ORIGINACION PARA OBTENER LA INFORAMCION DE ESTAS SOLICITUDES PASANDOLES EL ARRAY DE SOLICITUDES
        $solicitudes = $this->guzzle->findSolicitudes($solicitudes_comite);


        #RECORRER LOS ARRAY DE COMITE Y DE SCORIGN PARA UNIRLOS

        $solicitudes = collect($solicitudes);

        // $solicitudes->first()->calle = 20;

        foreach(collect($solicitudes) as $solicitud){
            #REALIZAR LA PETICION TIPO GET SCORING
            $solicitudes_scoring = $this->guzzle->findScoring($solicitud->id);
            // #convertir un objeto stdClass a array
            $solicitudes_scoring = json_decode(json_encode($solicitudes_scoring),true);

            if($this->validarScoringVacio($solicitudes_scoring) == 2){

                foreach($solicitud as $item){
                    $solicitudes->firstWhere('id', $solicitud->id)->valor_propiedad = $solicitudes_scoring['avatar']['comportamiento_economico'];
                    $solicitudes->firstWhere('id', $solicitud->id)->tiempo_comercializacion = $solicitudes_scoring['avatar']['antigüedad_negocio'];
                    $solicitudes->firstWhere('id', $solicitud->id)->score_buro_credito = $solicitudes_scoring['avatar']['probabilidad_fraude'];
                    $solicitudes->firstWhere('id', $solicitud->id)->calificacion_avatar = $solicitudes_scoring['avatar']['probabilidad_informacion_cuantitativa_falsa'];
                }

            }
        }
        
        return response()->json($solicitudes);

    }
    
    
    #valida que el array de Scoring no este vacio
    #return 1 -> Vacio
    #return 2 -> no Vacio
    public function validarScoringVacio($solicitudes_scoring){
        $count = 0;
        foreach($solicitudes_scoring as $solicitud){
            
                $val = $solicitud;
                if($val){
                    $count++;
                }
        }
        
        return $count;
    }
