# shopping_site_EKS_along_AI-ML_integration

Brought up my machine with sufficient memory and storage to (whatever 8 GB machine covered under the FREE Tier 😅) </br>
<img width="1622" height="804" alt="Image" src="https://github.com/user-attachments/assets/14380845-ae15-4bbc-8df0-1c2c13292286" />

---

Then tried connecting to it using my **WSL** terminal but it was not connecting.
<img width="899" height="41" alt="Image" src="https://github.com/user-attachments/assets/a1151e0d-aa6e-49ec-a17c-962997143282" />

---

Then I thought maybe there was some issue with the security group as I had taken the default SG, and yes that was the issue:
<img width="1814" height="271" alt="Image" src="https://github.com/user-attachments/assets/5fe4b432-c4d9-4504-adbd-5d7b806da00e" />

---

So temporarily I gave it SSH permission from anywhere:
<img width="1814" height="522" alt="Image" src="https://github.com/user-attachments/assets/15157627-0409-4b37-b260-34d035021c51" />

---

And it worked but there was some issue with the permission of the pem file as general downloading of the pem file through AWS makes permission too open by default as we can see in the error message.
<img width="1053" height="269" alt="Image" src="https://github.com/user-attachments/assets/3b2386f7-66d0-4a8f-8174-06d361a666ec" />


---

So I changed the permission and tried reconnecting and finally it worked
<img width="1063" height="277" alt="Image" src="https://github.com/user-attachments/assets/f2c338b2-365d-4fdd-b7c8-78b83f7fd54a" />

