i. IP: 34.212.24.58<br />
PORT: 2200<br />
USERNAME: grader<br />
ii. 34.212.24.58/itemcatalog<br />
iii. The pip packages installed include: sqlalchemy, requests, oauth2client, and Flask.<br />
Configurations include: User grader has super user permissions and can use sudo command, SSH key created and enforced, SSH is hosted on non default port, only allows connections for SSH(port 2200), HTTP(port 80), and NTP(port 123), and Item Catalog application hosted as a WSGI app.<br />
iv. SSH key is the 'grader_key.pem' file located within this repo.<br />
v. http://fosshelp.blogspot.in/2014/03/how-to-deploy-flask-application-with.html<br />
This was used to help direct me towards deploying my flask Item Catalog app.
