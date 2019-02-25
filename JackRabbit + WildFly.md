#Configuration

*	
	Download JCR (https://download.oracle.com/otndocs/jcp/content_repository-2.0-fr-oth-JSpec/) extract jcr-2.0.jar
	Add module to wildfly : 
		
		module.xml
		<?xml version="1.0" encoding="UTF-8"?>
		<module xmlns="urn:jboss:module:1.5" name="javax.jcr">
			<dependencies>
				<module name="javax.transaction.api" export="true"/>
			</dependencies>
			<resources>
				<resource-root path="jcr-2.0.jar"/>
			</resources>
		</module>

*
	Create user and DB in Posgresql.
	
		create database jackrabbit;
		create role jackrabbit with login password 'rfybcnhf';
		grant all privileges on schema public to jackrabbit;
	
	Name = jackrabbi, login = jackrabbi, password = jackrabbi

*
	Create folder jackrabbi in jboss.home.dir/standalone/data and place repository.xml in it.

		repository.xml
		<?xml version="1.0" encoding="UTF-8"?>
		<!--
		   Licensed to the Apache Software Foundation (ASF) under one or more
		   contributor license agreements.  See the NOTICE file distributed with
		   this work for additional information regarding copyright ownership.
		   The ASF licenses this file to You under the Apache License, Version 2.0
		   (the "License"); you may not use this file except in compliance with
		   the License.  You may obtain a copy of the License at
		 
			   http://www.apache.org/licenses/LICENSE-2.0
		 
		   Unless required by applicable law or agreed to in writing, software
		   distributed under the License is distributed on an "AS IS" BASIS,
		   WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
		   See the License for the specific language governing permissions and
		   limitations under the License.
		-->
		<!DOCTYPE Repository PUBLIC "-//The Apache Software Foundation//DTD Jackrabbit 1.5//EN" "http://jackrabbit.apache.org/dtd/repository-1.5.dtd">
		<!-- Example Repository Configuration File
			 Used by
			 - org.apache.jackrabbit.core.config.RepositoryConfigTest.java
			 -
		-->
		<Repository>
			<!--
				virtual file system where the repository stores global state
				(e.g. registered namespaces, custom node types, etc.)
			-->
			<FileSystem class="org.apache.jackrabbit.core.fs.local.LocalFileSystem">
				<param name="path" value="${rep.home}/repository"/>
			</FileSystem>
		 
			<!--
				security configuration
			-->
			<Security appName="Jackrabbit">
				<!--
					security manager:
					class: FQN of class implementing the JackrabbitSecurityManager interface
				-->
				<SecurityManager class="org.apache.jackrabbit.core.security.simple.SimpleSecurityManager" workspaceName="security">
					<!--
					workspace access:
					class: FQN of class implementing the WorkspaceAccessManager interface
					-->
					<!-- <WorkspaceAccessManager class="..."/> -->
					<!-- <param name="config" value="${rep.home}/security.xml"/> -->
				</SecurityManager>
		 
				<!--
					access manager:
					class: FQN of class implementing the AccessManager interface
				-->
				<AccessManager class="org.apache.jackrabbit.core.security.simple.SimpleAccessManager">
					<!-- <param name="config" value="${rep.home}/access.xml"/> -->
				</AccessManager>
		 
				<LoginModule class="org.apache.jackrabbit.core.security.simple.SimpleLoginModule">
				   <!--
					  anonymous user name ('anonymous' is the default value)
					-->
				   <param name="anonymousId" value="anonymous"/>
				   <!--
					  administrator user id (default value if param is missing is 'admin')
					-->
				   <param name="adminId" value="admin"/>
				</LoginModule>
			</Security>
		 
			<!--
				location of workspaces root directory and name of default workspace
			-->
			<Workspaces rootPath="${rep.home}/workspaces" defaultWorkspace="default"/>
			<!--
				workspace configuration template:
				used to create the initial workspace if there's no workspace yet
			-->
			<Workspace name="${wsp.name}">
				<!--
					virtual file system of the workspace:
					class: FQN of class implementing the FileSystem interface
				-->
				<FileSystem class="org.apache.jackrabbit.core.fs.local.LocalFileSystem">
					<param name="path" value="${wsp.home}"/>
				</FileSystem>
				<!--
					persistence manager of the workspace:
					class: FQN of class implementing the PersistenceManager interface
				-->
		 
				<PersistenceManager class="org.apache.jackrabbit.core.persistence.pool.PostgreSQLPersistenceManager">
						<param name="driver" value="org.postgresql.Driver"/>
						<param name="url" value="jdbc:postgresql://localhost:5432/jackrabbit"/>
						<param name="databaseType" value="postgresql"/>
						<param name="user" value="jackrabbit"/>
						<param name="password" value="jackrabbit"/>
						<param name="schemaObjectPrefix" value="${wsp.name}_"/>
						<param name="externalBLOBs" value="false"/>
				</PersistenceManager>
		 
				<!--
					Search index and the file system it uses.
					class: FQN of class implementing the QueryHandler interface
				-->
				<SearchIndex class="org.apache.jackrabbit.core.query.lucene.SearchIndex">
					<param name="path" value="${wsp.home}/index"/>
					<param name="textFilterClasses" value="org.apache.jackrabbit.extractor.PlainTextExtractor,org.apache.jackrabbit.extractor.MsWordTextExtractor,org.apache.jackrabbit.extractor.MsExcelTextExtractor,org.apache.jackrabbit.extractor.MsPowerPointTextExtractor,org.apache.jackrabbit.extractor.PdfTextExtractor,org.apache.jackrabbit.extractor.OpenOfficeTextExtractor,org.apache.jackrabbit.extractor.RTFTextExtractor,org.apache.jackrabbit.extractor.HTMLTextExtractor,org.apache.jackrabbit.extractor.XMLTextExtractor"/>
					<param name="extractorPoolSize" value="2"/>
					<param name="supportHighlighting" value="true"/>
				</SearchIndex>
			</Workspace>
		 
			<!--
				Configures the versioning
			-->
			<Versioning rootPath="${rep.home}/version">
				<!--
					Configures the filesystem to use for versioning for the respective
					persistence manager
				-->
				<FileSystem class="org.apache.jackrabbit.core.fs.local.LocalFileSystem">
					<param name="path" value="${rep.home}/version"/>
				</FileSystem>
		 
				<!--
					Configures the persistence manager to be used for persisting version state.
					Please note that the current versioning implementation is based on
					a 'normal' persistence manager, but this could change in future
					implementations.
				-->
				<PersistenceManager class="org.apache.jackrabbit.core.persistence.pool.PostgreSQLPersistenceManager">
						<param name="driver" value="org.postgresql.Driver"/>
						<param name="url" value="jdbc:postgresql://localhost:5432/jackrabbit"/>
						<param name="databaseType" value="postgresql"/>
						<param name="user" value="jackrabbit"/>
						<param name="password" value="jackrabbit"/>
						<param name="schemaObjectPrefix" value="version_"/>
						<param name="externalBLOBs" value="false"/>
				</PersistenceManager>
			</Versioning>
		 
			<!--
				Search index for content that is shared repository wide
				(/jcr:system tree, contains mainly versions)
			-->
			<SearchIndex class="org.apache.jackrabbit.core.query.lucene.SearchIndex">
				<param name="path" value="${rep.home}/repository/index"/>
				<param name="textFilterClasses" value="org.apache.jackrabbit.extractor.PlainTextExtractor,org.apache.jackrabbit.extractor.MsWordTextExtractor,org.apache.jackrabbit.extractor.MsExcelTextExtractor,org.apache.jackrabbit.extractor.MsPowerPointTextExtractor,org.apache.jackrabbit.extractor.PdfTextExtractor,org.apache.jackrabbit.extractor.OpenOfficeTextExtractor,org.apache.jackrabbit.extractor.RTFTextExtractor,org.apache.jackrabbit.extractor.HTMLTextExtractor,org.apache.jackrabbit.extractor.XMLTextExtractor"/>
				<param name="extractorPoolSize" value="2"/>
				<param name="supportHighlighting" value="true"/>
			</SearchIndex>
		</Repository>

* 
	Configurate resource adapter : 
	
		Standalone.xml 
		into xmlns="urn:jboss:domain:resource-adapters:5.0" 
		add next code
			<resource-adapters>
				<resource-adapter id="jackrabbit-jca-2.16.2.rar">
					<archive>
						jackrabbit-jca-2.16.2.rar
					</archive>
					<connection-definitions>
						<connection-definition class-name="org.apache.jackrabbit.jca.JCAManagedConnectionFactory" jndi-name="jcr/local" enabled="true" pool-name="RabbitAdapter">
							<config-property name="homeDir">
								${jboss.home.dir}/standalone/data/jackrabbit
							</config-property>
							<config-property name="configFile">
								${jboss.home.dir}/standalone/data/jackrabbit/repository.xml
							</config-property>
							<security>
								<application/>
							</security>
							<validation>
								<background-validation>false</background-validation>
							</validation>
						</connection-definition>
					</connection-definitions>
				</resource-adapter>
			</resource-adapters>
*
	Turn off archive validation
	
		Standalone.xml 
		<subsystem xmlns="urn:jboss:domain:jca:5.0">
			<archive-validation enabled="false" fail-on-error="true" fail-on-warn="false"/>
		</subsystem>
*
	Download jackrabbit-jca-2.16.2.rar (https://jackrabbit.apache.org/jcr/downloads.html)
	Open  jackrabbit-jca-2.16.2/META-INF/MANIFEST.MF and add 
	Dependencies: javax.jcr export,org.slf4j,org.postgresql
* 
	Deploy jackrabbit-jca-2.16.2.rar (copy it into jboss.home.dir/standalone/deployments)
*
	Add	into jboss-deployment-structure.xml 	
	
		<module name="deployment.jackrabbit-jca-2.16.2.rar"/>  
