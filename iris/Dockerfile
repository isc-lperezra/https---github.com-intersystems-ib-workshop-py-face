ARG IMAGE=containers.intersystems.com/intersystems/iris-community:2023.1.0.229.0
FROM $IMAGE

USER root

WORKDIR /opt/irisapp
RUN chown -R irisowner:irisowner /opt/irisapp

RUN apt-get update && apt-get install -y python3

RUN apt-get update && \
    apt-get install -y libgl1-mesa-glx libglib2.0-0

# install required packages
COPY --chown=$ISC_PACKAGE_MGRUSER:$ISC_PACKAGE_IRISGROUP /requirements.txt /
RUN chmod 777 /requirements.txt

RUN pip3 install -r /requirements.txt

USER irisowner

# copy files to image
WORKDIR /opt/irisapp
#COPY --chown=irisowner:irisowner /iris/iris.script iris.script
COPY --chown=irisowner:irisowner src src

COPY --chown=$ISC_PACKAGE_MGRUSER:$ISC_PACKAGE_IRISGROUP /iris/irissession.sh /
RUN chmod +x /irissession.sh

SHELL ["/irissession.sh"]

RUN \
  zn "USER" \
  # do ##class(Ens.Config.Credentials).SetCredential("MySQL_User","root","SYS") \
  # load & compile source code
  set sc = $SYSTEM.OBJ.LoadDir("/opt/irisapp/src/Installer", "ck", , 1) \
  if sc do ##class(Nomenclator.Installer).Install() \
  zn "NOMENCLATOR" \
  set sc = $SYSTEM.OBJ.LoadDir("/opt/irisapp/src/Nomenclator", "ck", , 1) \
  # do ##class(%File).RemoveDirectoryTree("/tmp") \
  # start production
  set production = "Nomenclator.Production" \
  set ^Ens.Configuration("csp","LastProduction") = production \
  do ##class(Ens.Director).SetAutoStart(production)
  
# run iris.script
# RUN iris start IRIS \
#   && iris session IRIS < /opt/irisapp/iris.script \
#   && iris stop IRIS quietly