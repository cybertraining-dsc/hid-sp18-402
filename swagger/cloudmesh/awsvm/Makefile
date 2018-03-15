#swagger code gen path
SWAGGER_CODEGEN=java -jar swagger-codegen-cli-2.3.1.jar
SPEC_FILE=aws_vm.yml
DESTDIR=awsserver
#requires mongodb installed
MONGO_DB_PATH=~/mongo/data/db

help: ## show this help
	@echo 'usage: make [target] ...'
	@echo 'Make file sequence - startdb, service, test, stop'
	@echo 'targets:'
	@egrep '^(.+)\:\ .*##\ (.+)' ${MAKEFILE_LIST} | sed 's/:.*##/#/' | column -t -c 2 -s '#'

.PHONY : init clean

init:
	mkdir $(DESTDIR)

service: clean init ## generate service and start the swagger server
	echo "Generating swagger code"
	$(SWAGGER_CODEGEN) generate \
		-i $(SPEC_FILE) \
		-l python-flask \
		-o $(DESTDIR) \
		-D supportPython2=true
	cp -r *.py $(DESTDIR)/swagger_server/controllers/
	echo "Creating virtualenv to run swagger server"
	#create virtualenv
	virtualenv venv
	. venv/bin/activate
	#cd $(DESTDIR) && venv/bin/pip install -r requirements.txt && \
        #venv/bin/python setup.py install
	venv/bin/python -m pip install pymongo
	echo "Staring Swagger server"
	cd $(DESTDIR) && ../venv/bin/pip install -r requirements.txt && \
        ../venv/bin/python setup.py install && ../venv/bin/python -m swagger_server

startdb: ## start MangoDB
	echo "Starting MongoDB"
	mongod --dbpath $(MONGO_DB_PATH)

test: ## Test swagger service
	echo "Create VM -->"
	curl -X POST "http://localhost:8080/aws/vm" -H  "accept: application/json" -H  "Content-Type: application/json" -d "{  \"vmId\": 5,  \"name\": \"aws vm 5\",  \"image\": \"linux\",  \"location\": \"us\",  \"ramSize\": 2020,  \"diskSize\": 101010,  \"status\": \"RUNNING\"}"
	echo "Get VM List-->"
	curl -X GET "http://localhost:8080/aws/vms" -H  "accept: application/json"
	echo "Get VM by ID-->"
	curl -X GET "http://localhost:8080/aws/vms/1" -H  "accept: application/json"
stop: ## Stop swagger server and database
	echo "Stopping swagger serevr"
	ps -ef | grep 'swagger_server' | grep -v grep | awk '{print $$2}' | xargs kill
	echo "Stopping Database MongoDB"
	mongod --dbpath $(MONGO_DB_PATH) --shutdown

clean: ## Clean generated directories
	echo "Removing directories"
	rm -r $(DESTDIR)
	rm -r venv
