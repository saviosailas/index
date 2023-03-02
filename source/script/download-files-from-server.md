<h1>wget script to download files from server</h1>

1. Download all files <br>
<code>wget -r --no-parent http://192.168.0.1:8090/localfile</code>

2. Exclude index.html <br>
<code>wget -r --no-parent --reject "index.html*" http://192.168.0.1:8090/localfile</code> <br>
<code>wget -r --no-parent -R "index.html*" http://192.168.0.1:8090/localfile</code>

3. Bypass robot.txt <br> 
<code>wget -e robots=off http://192.168.0.1:8090/localfile</code>

4. Mirror website <br> 
<code>wget -m http://192.168.0.1:8090/localfile</code>

5. Use of user agent <br> 
<code>wget -r --no-parent -R "index.html*" --user-agent=Mozilla/5.0 http://192.168.0.1:8090/localfile</code>

6. Server with authentication <br>
<code>wget -r --user=USER_NAME --password='(PASSWORD' http://192.168.0.1:8090/localfile</code>

7. Include all webpage resources like JavaScripts and other files <br>
<code>wget -r -p --no-parent http://192.168.0.1:8090/localfile</code> <br>
<code>wget -rp --no-parent http://192.168.0.1:8090/localfile</code> 

8. Convert links to localfiles <br>
<code>wget -r -k --no-parent http://192.168.0.1:8090/localfile</code> <br>
<code>wget -rk --no-parent http://192.168.0.1:8090/localfile</code>

9. Download only if remote files modified date is not older than local files <br>
<code>wget -r -N --no-parent http://192.168.0.1:8090/localfile</code> <br>
<code>wget -rN --no-parent http://192.168.0.1:8090/localfile</code>

10. Remuse download if connection goes down <br>
<code>wget -r -c --no-parent http://192.168.0.1:8090/localfile</code> <br>
<code>wget -rc --no-parent http://192.168.0.1:8090/localfile</code> 

11. Remove host url folder <br>
<code>wget -r --cut-dirs=2 --no-parent http://192.168.0.1:8090/localfile</code> <br>

<h5> For more info go through <a href="http://www.gnu.org/software/wget/manual/html_node/"> wget</a> manuel or download <a href="http://www.gnu.org/software/wget/manual/wget.pdf"> pdf</a> </h5>
