    public function index(Request $request){
#REALIZAR LA PETICION TIPO GET A SERVICIOS-ORIGINACION PARA OBTENER LA INFORAMCION DE ESTAS SOLICITUDES PASANDOLES EL ARRAY DE SOLICITUDES
        $solicitudes = $this->guzzle->findSolicitudes($solicitudes_comite);


        #RECORRER LOS ARRAY DE COMITE Y DE SCORIGN PARA UNIRLOS
        $solicitudes = collect($solicitudes);

        foreach($solicitudes as $solicitud){
            #REALIZAR LA PETICION TIPO GET SCORING
            $solicitudes_scoring = $this->guzzle->findScoring($solicitud->id);
            // #convertir un objeto stdClass a array
            $solicitudes_scoring = json_decode(json_encode($solicitudes_scoring),true);

            if($this->validarScoringVacio($solicitudes_scoring) == 2){

                    $solicitudes->firstWhere('id', $solicitud->id)->valor_propiedad = $solicitudes_scoring['avatar']['comportamiento_economico'];
                    $solicitudes->firstWhere('id', $solicitud->id)->tiempo_comercializacion = $solicitudes_scoring['avatar']['antigüedad_negocio'];
                    $solicitudes->firstWhere('id', $solicitud->id)->score_buro_credito = $solicitudes_scoring['avatar']['probabilidad_fraude'];
                    $solicitudes->firstWhere('id', $solicitud->id)->calificacion_avatar = $solicitudes_scoring['avatar']['probabilidad_informacion_cuantitativa_falsa'];

            }
        }
        
        return response()->json($solicitudes);
