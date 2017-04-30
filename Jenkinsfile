node( 'maven' ) {


    stage('checkout'){
        checkout scm
    }

    stage('put work on queue'){
        login()
        sh 'oc apply -f job-producer.yaml'
        jobComplete("producer-job")
    }



}

def login() {
    sh """
       set +x
       oc login --certificate-authority=/var/run/secrets/kubernetes.io/serviceaccount/ca.crt --token=\$(cat /var/run/secrets/kubernetes.io/serviceaccount/token) https://kubernetes.default.svc.cluster.local >/dev/null 2>&1 || echo 'OpenShift login failed'
       """
}

def jobComplete(String jobName) {

    sh """
      set +x
      COUNTER=0
      DELAY=10
      MAX_COUNTER=30
      echo "checking job ${jobName} for completion"
      set +e
      while [ \$COUNTER -lt \$MAX_COUNTER ]
      do
        JOB_RESPONSE=\$(oc get job  ${jobName} --template="{{.status.conditions}}")
        echo "\$JOB_RESPONSE" | grep "type: Complete" >/dev/null 2>&1
        if [ \$? -eq 0 ]; then
          echo "Job Succeeded!"
          break
        fi
        if [ \$COUNTER -lt \$MAX_COUNTER ]; then
          echo -n "."
          COUNTER=\$(( \$COUNTER + 1 ))
        fi
        if [ \$COUNTER -eq \$MAX_COUNTER ]; then
          echo "Max Validation Attempts Exceeded. Failed Verifying Job Completion..."
          exit 1
        fi
        sleep \$DELAY
      done
      set -e
    """

}
