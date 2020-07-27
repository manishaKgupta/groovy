# groovy
Groovy code 

job ("Job1-Groovy") {
	
  description ("Job1 to pull GitHub Repo")
  
  scm {
    github('manishaKgupta/webserver','master')
       }
  
  triggers {
	scm("* * * * * ")
      }
}

job ("Job2-Groovy") {
	
  description ("Job2 to launch the pods")
  triggers{
			upstream('Job1-Groovy' , 'SUCCESS')
		}
  
  scm {
    github('manishaKgupta/webserver','master')
       }
  
  triggers {
	scm("* * * * * ")
      }
  steps {
  shell(''' sudo cp index.html /root/jenkins
if sudo docker images | grep vimal13/apache-webserver-php
then
echo "Already exists"
else
sudo docker pull vimal13/apache-webserver-php
fi
if sudo kubectl get deployment | grep mytask3-deploy
then 
echo "Already Exists"
else
sudo kubectl create -f /root/deploy-pvc.yml
sudo kubectl expose deployment mytask3-deploy  --port=80 --type=NodePort
sudo kubectl get pods -o go-template --template '{{range .items}}{{.metadata.name}}{{"\n"}}{{end}}' > file
pod_name=$(cat file)
sleep 5
sudo kubectl cp /root/index.html $pod_name:/var/www/html/index.html ~/index.html 
fi '''

)
  }
}

job ("Job3-Groovy") {
	
  description ("Job3 for Testing the webpage")
  triggers{
			upstream('Job2-Groovy' , 'SUCCESS')
		}
  publishers {
        mailer('guptamanisha3845@gmail.com')
    }
  
  scm {
    github('manishaKgupta/webserver','master')
       }
  
  triggers {
	scm("* * * * * ")
      }
steps {
  shell('''sudo kubectl get svc --all-namespaces -o go-template='{{range .items}}{{range.spec.ports}}{{if .nodePort}}{{.nodePort}}{{"\n"}}{{end}}{{end}}{{end}}' > port-no
sudo cat port-no
port=$(cat port-no)
status=$(curl -o /dev/null -s -w "%{http_code}" http://192.168.99.105:$port)
if [[ $status == 200 ]]
then 
exit 0
else
exit 1
fi ''')
}

 publishers {
        extendedEmail {
            recipientList('guptamanisha3845@gmail.com')
            defaultSubject('Oops')
            defaultContent('Something broken')
            contentType('text/html')
            attachBuildLog(true)
            triggers {
                beforeBuild()
                stillUnstable {
                    subject('CI/CD')
                    content('Your build is still unstable')
                    sendTo {
                        developers()
                        requester()
                        culprits()
                    }
                }
            }
        }
    }

}
buildPipelineView('TASK6-DEVOPS'){
filterBuildQueue(true)
filterExecutors(false)
title('TASK-3')
displayedBuilds(5)
selectedJob('Job1-Groovy')
alwaysAllowManualTrigger(false)
showPipelineParameters(true)
refreshFrequency(1)
}





