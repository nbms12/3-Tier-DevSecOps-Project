Day 1 Goal : setup local prject and verify 


steps : 

1.clone project url 

2. install node in local sysytem

```
# Download and install nvm:
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.40.3/install.sh | bash
# in lieu of restarting the shell
\. "$HOME/.nvm/nvm.sh"
# Download and install Node.js:
nvm install 22
# Verify the Node.js version:
node -v # Should print "v22.16.0".
nvm current # Should print "v22.16.0".
# Verify npm version:
npm -v # Should print "10.9.2".

```

3. install dependencies of backend ( ie under API folder )

   use > npm install


4. install dependencies of frontend ( ie under client folder )

   npm install


5. install mysql ( used docx file instructions  )

6.update .env file api url ( under client folder ) tis will endpoint interacts to frontend of app 

7.add ports for security groups ( 5000, 3000 ) 

8. start local build

   ```
   npm start
   ```

8. verify wit public ip and port 3000

   ![image](https://github.com/user-attachments/assets/690f766d-1f77-4bc5-b91b-2f3e6cb8df20)
