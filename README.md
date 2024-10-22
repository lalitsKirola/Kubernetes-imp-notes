# Kubernetes-imp-notes

![image](https://github.com/user-attachments/assets/fc7dd686-6fd1-4dd4-b5c4-3824acd261c2)


![image](https://github.com/user-attachments/assets/f69fd1e5-d9bb-4a9b-ab99-73b2fcf75943)

---In the above example , we will create two deployment pods and two corresponding services and above these two service , there will be a ingress service where we will
mention routing rules (path based routing or host based routing) .

---We will hit a url like ip/aksdemo1 , it will go to the ingress controller and then to ingress service , 
according to the rules in the ingress service it will reach to that service and then that associated pods.

