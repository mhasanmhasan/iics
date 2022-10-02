import json
import logging
from urllib.error import HTTPError
import requests
import getopt
from time import strftime
import logging.handlers as handlers
import os
import smtplib
from email.message import EmailMessage
import mimetypes
from email.mime.multipart import MIMEMultipart
from email import encoders
from email.message import Message
from email.mime.audio import MIMEAudio
from email.mime.base import MIMEBase
from email.mime.image import MIMEImage
from email.mime.text import MIMEText
from cryptography.fernet import Fernet
from time import sleep
import datetime

def sendemail():
    pass

def getprops():
    pass

def sessiontoken(*args):
    loginuri = baseURI + "v3/login"
    session = requests.Session()
    loginresp = session.post(loginuri, json=args[0])
    try:
        loginresp.raise_for_status()
    except HTTPError as e:
        print(str(e))
        raise
    INFASESSIONID = loginresp.json()['userInfo']['sessionId']
    session.headers.update({'INFA-SESSION-ID': INFASESSIONID})
    return session

def getobjectid(*args):
    getobjecturi = baseURI + "v3/objects"
    objectresponse = args[0].get(getobjecturi, params=args[1])
    try:
        objectresponse.raise_for_status()
    except HTTPError as e:
        print(str(e))
        raise
    return objectresponse.json()['objects'][0]['id']

def startexport(*args):
    startexporturi = baseURI + "v3/export"
    jobidresponse = args[0].post(startexporturi, json=
{"name" : args[2], "objects" : [{"id": args[1], "includeDependencies" : "true"}]})
    try:
        jobidresponse.raise_for_status()
    except HTTPError as e:
        print(str(e))
        raise
    return jobidresponse.json()['id']
          
def getexportstatus(*args):
    getexporturi = baseURI + "v3/export/" + args[1]
    statusresponse = args[0].get(getexporturi)
    try:
        statusresponse.raise_for_status()
    except HTTPError as e:
        print(str(e))
        raise
    return statusresponse.json()['status']['state']

def getexportlog(*args):
    getexporturi = baseURI + "v3/export/" + args[1] + "/log"
    getexportlogresponse = args[0].get(getexporturi)
    try:
        getexportlogresponse.raise_for_status()
    except HTTPError as e:
        print(str(e))
        raise
    return getexportlogresponse.text

def getpackage(*args):
    getpackageuri = baseURI + "v3/export/" + args[1] + "/package"
    getpackageresponse = args[0].get(getpackageuri)
    try:
        getpackageresponse.raise_for_status()
    except HTTPError as e:
        print(str(e))
        raise
    return getpackageresponse.content

def main():

    global baseURI
    baseURI = "<iics_baseURI>/saas/public/core/"
    iics_creds = dict(username='<xxxxx>',password='<xxxxx>')
    location = "'<iics_folder>'"
    objectdataform = "q=location=="+location+" and type=='Project'"
    jobname = "<name of export job>"
    filepath = "<abs. local path where export log and zip will be downloaded>"
    now = datetime.datetime.now()
    timestamp = str(now.strftime("%Y%m%d_%H-%M-%S"))
    exportlog = filepath + "/export-" + timestamp + ".log"
    exportzip = filepath + "/download-" + timestamp + ".zip"

    sess = sessiontoken(iics_creds)
    objid = getobjectid(sess, objectdataform)
    jobid = startexport(sess, objid, jobname)

    while getexportstatus(sess, jobid) != "SUCCESSFUL":
        sleep(5)

    try:
        with open(exportlog,'w+') as f:
            f.write(getexportlog(sess, jobid))
    except Exception as e:
        raise
    f.close()

    try:
        with open(exportzip,'wb+') as f:
            f.write(getpackage(sess, jobid))
    except Exception as e:
        raise
    f.close()

if __name__ == "__main__":
    main()

