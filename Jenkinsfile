import jenkins.model.*
import com.cloudbees.plugins.credentials.*
import com.cloudbees.plugins.credentials.domains.*
import org.jenkinsci.plugins.plaincredentials.impl.*
import hudson.util.Secret
import hudson.security.ACL
import hudson.security.ACLContext
import groovy.json.JsonSlurper

Thread.start {
    println "==> [Auto-Init] Waiting for SonarQube service to become fully initialized..."
    
    boolean sonarReady = false
    int retries = 0
    
    while (!sonarReady && retries < 50) {
        try {
            def process = ["sh", "-c", "curl -s http://10.26.0.198:9000/api/system/status"].execute()
            process.waitFor()
            def response = process.text
            if (response.contains('"status":"UP"')) {
                sonarReady = true
                println "==> [Auto-Init] SonarQube container API is UP!"
            }
        } catch (Exception e) {}
        
        if (!sonarReady) {
            retries++
            sleep(10000)
        }
    }

    if (sonarReady) {
        // Give SonarQube extra time to initialize internal DB user indices
        sleep(15000)
        
        println "==> [Auto-Init] Generating SonarQube token via API..."
        
        // FIXED: SonarQube uses 'pName' for user token names in modern versions
        String tokenName = "jenkins-auto-token"
        String cmd = "curl -s -v -u admin:admin -X POST 'http://10.26.0.198:9000/api/user_tokens/generate?pName=${tokenName}'"
        def genProc = ["sh", "-c", cmd].execute()
        genProc.waitFor()
        
        String jsonResp = genProc.text
        println "==> [Auto-Init] SonarQube Raw Response: ${jsonResp}"
        
        try {
            def jsonSlurper = new JsonSlurper()
            def obj = jsonSlurper.parseText(jsonResp)
            // Modern SonarQube returns token details under obj.token
            String tokenValue = obj.token

            if (tokenValue) {
                println "==> [Auto-Init] Token parsed successfully. Storing in Jenkins credentials..."

                ACLContext context = ACL.as2(ACL.SYSTEM2)
                try {
                    def store = Jenkins.instance.getExtensionList('com.cloudbees.plugins.credentials.SystemCredentialsProvider')[0].getStore()
                    def domain = Domain.global()
                    
                    def existing = store.getCredentials(domain).find { it.id == 'sonar-token' }
                    if (existing != null) {
                        store.removeCredentials(domain, existing)
                    }

                    def credential = new StringCredentialsImpl(
                        CredentialsScope.GLOBAL,
                        "sonar-token",
                        "Automated SonarQube Token",
                        Secret.fromString(tokenValue)
                    )
                    
                    store.addCredentials(domain, credential)
                    println "==> [Auto-Init] Credential 'sonar-token' created successfully!"
                } finally {
                    context.close()
                }
            } else {
                println "==> [Auto-Init] Error: 'token' field missing or invalid credentials response."
            }
        } catch (Exception parseEx) {
            println "==> [Auto-Init] Failed to parse JSON response."
            parseEx.printStackTrace()
        }
    } else {
        println "==> [Auto-Init] Timed out waiting for SonarQube service."
    }
}
