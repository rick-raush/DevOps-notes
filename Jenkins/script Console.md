### To get all users

  
import jenkins.model.\*  
import hudson.model.\*

Jenkins.instance.getAllItems().each { } // ensures Jenkins context loads

def users = User.getAll()

users.each { user ->  
    def email = user.getProperty(hudson.tasks.Mailer.UserProperty)  
    println "\${user.getId()} : \${email?.getAddress()}"  
}

* * *

&nbsp;