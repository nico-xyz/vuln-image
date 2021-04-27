  - name: trivy
      image: docker.io/aquasec/trivy
      tty: true
      command: ["/bin/sh"]
      workingDir: ${workingDir}
      securityContext:
        privileged: true
      envFrom:
        - configMapRef:
            name: ibmcloud-config
        - secretRef:
            name: ibmcloud-apikey
            container(name: 'buildah', shell: '/bin/bash') {
                stage('Build image') {
                    sh '''#!/bin/bash
                        set -e
                        . ./env-config

    		            echo TLSVERIFY=${TLSVERIFY}
    		            echo CONTEXT=${CONTEXT}
                    IMAGE_VERSION="0.0.1"

    		            if [[ -z "${REGISTRY_PASSWORD}" ]]; then
    		              REGISTRY_PASSWORD="${APIKEY}"
    		            fi

                    APP_IMAGE="${REGISTRY_URL}/${REGISTRY_NAMESPACE}/${IMAGE_NAME}-temp:${IMAGE_VERSION}"

                    echo "Image... $APP_IMAGE"

                    buildah bud --tls-verify=${TLSVERIFY} --format=docker -f ${DOCKERFILE} -t ${APP_IMAGE} ${CONTEXT}

                    if [[ -n "${REGISTRY_USER}" ]] && [[ -n "${REGISTRY_PASSWORD}" ]]; then
                        buildah login -u "${REGISTRY_USER}" -p "${REGISTRY_PASSWORD}" "${REGISTRY_URL}"
                    fi

                    buildah push --tls-verify=${TLSVERIFY} "${APP_IMAGE}" "docker://${APP_IMAGE}"
                    '''
                }
            }
            container(name: 'trivy', shell: '/bin/sh') {
                stage('Trivy Scan') {

                      sh '''#!/bin/sh

                        set -e
                        . ./env-config

                        echo TLSVERIFY=${TLSVERIFY}
                        echo CONTEXT=${CONTEXT}
                        IMAGE_VERSION="0.0.1"

                        if [[ -z "${REGISTRY_PASSWORD}" ]]; then
                          REGISTRY_PASSWORD="${APIKEY}"
                        fi

                        APP_IMAGE_Temp="${REGISTRY_URL}/${REGISTRY_NAMESPACE}/${IMAGE_NAME}-temp:${IMAGE_VERSION}"

                        export TRIVY_AUTH_URL=${REGISTRY_URL}
                        export TRIVY_USERNAME=${REGISTRY_USER}
                        export TRIVY_PASSWORD=${REGISTRY_PASSWORD}

                        echo "Trivy image scanning started.... $APP_IMAGE_Temp"

                        # Trivy scan
                        trivy ${APP_IMAGE_Temp}
                        # trivy --exit-code 1 --severity CRITICAL ${APP_IMAGE_Temp}

                        # Trivy scan result processing
                        my_exit_code=$?
                        echo "RESULT 1:--- $my_exit_code"

                        # Check scan results
                        if [ ${my_exit_code} == 1 ]; then
                            echo "Image scanning failed. Some vulnerabilities found"
                            exit 1;
                        else
                            echo "Image is scanned Successfully. No vulnerabilities found"
                        fi;
                    '''
                }
            }
            container(name: 'buildah', shell: '/bin/bash') {
                 stage('Push image') {
                     sh '''#!/bin/bash
                         set -e
                         . ./env-config

     		            echo TLSVERIFY=${TLSVERIFY}
     		            echo CONTEXT=${CONTEXT}
                     IMAGE_VERSION="0.0.1"

     		            if [[ -z "${REGISTRY_PASSWORD}" ]]; then
     		              REGISTRY_PASSWORD="${APIKEY}"
     		            fi

                     APP_IMAGE="${REGISTRY_URL}/${REGISTRY_NAMESPACE}/${IMAGE_NAME}:${IMAGE_VERSION}"
                     APP_IMAGE_Temp="${REGISTRY_URL}/${REGISTRY_NAMESPACE}/${IMAGE_NAME}-temp:${IMAGE_VERSION}"

                     # #buildah bud --tls-verify=${TLSVERIFY} --format=docker -f ${DOCKERFILE} -t ${APP_IMAGE} ${CONTEXT}
                     if [[ -n "${REGISTRY_USER}" ]] && [[ -n "${REGISTRY_PASSWORD}" ]]; then
                         buildah login -u "${REGISTRY_USER}" -p "${REGISTRY_PASSWORD}" "${REGISTRY_URL}"
                     fi
                     # buildah push --tls-verify=${TLSVERIFY} "${APP_IMAGE}" "docker://${APP_IMAGE}"
             
                     buildah pull ${APP_IMAGE_Temp}
                     buildah tag ${APP_IMAGE_Temp} ${APP_IMAGE}
                     buildah push --tls-verify=${TLSVERIFY} "${APP_IMAGE}" "docker://${APP_IMAGE}"
                     buildah rmi ${APP_IMAGE_Temp}
                     '''
                 }
             }  
