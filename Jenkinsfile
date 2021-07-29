#!/usr/bin/env groovy

import org.common.*
import hudson.model.*


String buildResultInformation = ""
boolean buildError = false
boolean buildUnstable = false

String slnFile = 'Mytestproject.sln'
String projFile = "Mytestproject\\Mytestproject.csproj "
String publishedPath = "Mytestproject\\Mytestproject.App\\bin\\Release\\netcoreapp2.2\\publish"

String conf = "Release"
String platform = "Any CPU"


String iisApplicationName = "Mytestproject "
String iisApplicationPath = "C:\\Users\rattuluri\\Desktop\\Mytestproject"
String targetServerIP = "0.0.0.0"


node("master") {
    try
    {
        stage('Cleanup') {
                step([$class: 'WsCleanup', cleanWhenFailure: true])
            }
            
        stage('Git clone') {
            git branch: 'master', url: 'https://github.com/raviravi89/mytestproject.git'
        }
        
        stage('Build') {
            bat """
            msbuild ${slnFile} /t:build /p:Configuration=${conf} /P:DeployOnBuild=true /p:BuildingSolutionFile=true """
        }

        stage("Stop IIS site") {
            powershell """
                    Stop-WebAppPool -Name "Mytestproject"

                """
        }

        stage("Application backup") {
            powershell """
                    Write-Output "Taking the application backup"
                    Copy-Item -Path "C:\\Users\\rattuluri\\Desktop\\Mytestproject" "C:\\Users\\rattuluri\\Desktop\\Mytestproject_bkp" -Force

                """
        }

        stage("Deploy new code"){
            def msDeploy = "C:\\Program Files (x86)\\IIS\\Microsoft Web Deploy V3\\msdeploy.exe"
            bat """ 
            "${msDeploy}" -verb:sync -source:iisApp="${WORKSPACE}\\${publishedPath}" -enableRule:AppOffline -dest:iisApp="${iisApplicationName}",ComputerName="https://${targetServerIP}:8172/msdeploy.axd", AuthType="Basic" -allowUntrusted"""
        }
        
    }
    
    catch (e)
    {
        echo "${e}"
        buildError = true
        buildResultInformation = "${e}"
        currentBuild.displayName = "Failed-build-" + BUILD_NUMBER
        currentBuild.result = 'FAILURE'
    }
    finally
    {
        emailext body: """
                        <p><strong>Build #${BUILD_NUMBER} is - <span style="color: rgb(235, 107, 86);">Failed</span></strong></p>
                        <p><strong>Output:</strong> ${BUILD_URL}</p>
                        <p><strong>Error Occurred: <span style="color: rgb(84, 172, 210);">&nbsp;</span><span style="color: rgb(235, 107, 86);">${buildResultInformation}</span></strong></p>
                        <p>-------------------------------------------------------</p>""",
                subject: 'Error at build server',
                to: 'xyz@abc.com'
    }
}