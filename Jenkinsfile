//Ejecucion Local
import java.text.SimpleDateFormat
import java.sql.Timestamp
import java.util.concurrent.TimeUnit
def dateFormat = new SimpleDateFormat("yyyyMMddHHmm")
def date = new Date()
def timestamp = dateFormat.format(date).toString()
pipeline
{
    options {
      timeout(time: 1, unit: 'HOURS') 
    }
    agent any

    stages {
        stage('Actualizar fuentes') {
            steps {
                echo 'Actualizar fuentes'
                checkout([$class: 'GitSCM', branches: [[name: "master"]], 
                doGenerateSubmoduleConfigurations: false, extensions: [], submoduleCfg: [], userRemoteConfigs: [
                [credentialsId: "opalacio1", url: "https://github.com/opalacioo/pruebaperfor.git"]
                ]])
            }
        }
        stage('Ejecutar prueba Baseline Orange') {
            steps {
                script {
                    try{
                        echo 'Performance Guardar Build And JobName' 
                        bat "C:\\apache-choucair\\bin\\jmeter -n -t -JJOB_NAME=Baseline_${env.BUILD_NUMBER}"
                        echo "Ejecutar Performance Test"
                        bat "bzt $WORKSPACE\\clima_cpt.yml $WORKSPACE\\passfail_config.yml"
                        perfReport './clima_cpt.xml'
                    } 
                    catch(ex)
                    {
                        echo 'Finalizo ejecucion con fallos'
                        bat "neoload.exe -project Error.npl"
                    }
                }
            }
        }
    }
    post {
        success {  
            enviarCorreo("SUCCESS")
        }  
        failure {  
            enviarCorreo("FAILURE")
        }
    }
}

def enviarCorreo(resultadoTest) {
    def asunto = ""
    def recibe = "kpava@choucairtesting.com,cvaldebenitoe@choucairtesting.com,cquiroz@choucairtesting"
    def resultado = ""
    def hero = ""
    def nombreProyecto = "Baseline Orange"

    def NOMBRE_TRANSACCION = "Demostracion CPT" //Nombre del proyecto para asunto del correo
    def NOMBRE_PROYECTO_GIT = "democpt" //Nombre del proyecto en GIT

    if (resultadoTest == "SUCCESS") {
        asunto = "CPT - EJECUCION EXITOSA ESCENARIO - TRANSACCION ${NOMBRE_TRANSACCION}"
        resultado = """<b class="execTitle" style="color:#032d9f;border:3px solid ;border-radius: 9px; padding:0px 12px 0 12px;">EJECUCION EXITOSA</b>"""
        hero = "Todo va bien en la transacci&oacute;n. Para m&aacute;s detalles remitase a los enlaces en la parte inferior de este correo."
    } else {
        asunto = "CPT - EJECUCION FALLIDA ESCENARIO - TRANSACCION ${NOMBRE_TRANSACCION}"
        resultado = """<b class="execTitle" style="color:#c90000;border:3px solid #c90000;border-radius: 9px; padding:0px 12px 0 12px;">EJECUCION FALLIDA</b>"""
        hero = "Se ha encontrado un error en la transacci&oacute;n. Para m&aacute;s detalles remitase a los enlaces en la parte inferior de este correo."
    }

    def body = "Ejuecucion CPT Exitosa"

    emailext(
        subject: "${asunto}",
        body: "${body}",
        to: "${recibe}"
    )
}
