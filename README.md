# Issue A Certificate on BlockChain Using BlockCerts(for Mac)

## Installation
1. **Python3**

  	Mac OS X High Sierra comes with Python2.7 by default, but we need Python3 in this project.
  	Here we use [HomeBrew](https://brew.sh/) to install Python3
        **Install homebrew**
        1. Open Terminal, install Apple's Xcode package first.

                xcode-select --install
	1. To install Homebrew, open Terminal and run

                /usr/bin/ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
                
	2. And then run the following command by adding the following line at the bottom of your ~/.bash_profile file
	
                echo "export PATH=/usr/local/bin:/usr/local/sbin:$PATH" >> ~/.bash_profile      && source ~/.bash_profile
                  
	1. install Python3

                brew install python3
                
  	
2. **virtualEnv & virtualEnvWrapper**  
	> virtualenv is a tool to create isolated Python environments. virtualenv creates a folder which contains all the necessary executables to use the packages that a Python project would need.  
	* Install virtualEnv via pip.  
		
		```
		pip install virtualenv
		```
	
	* Install virtualenvWrapper. 
		
		```
		mkdir ~/Envs
		pip install virtualenvwrapper
		```  
		Here, **~/Projects** is the project root directory
		
		```
		cat "export WORKON_HOME=~/Envs" >> ~/.bash_profile
		cat "export PROJECT_HOME=~/Projects" >> ~/.bash_profile
		cat "source /usr/local/bin/virtualenvwrapper.sh" >> ~/.bash_profile
		source ~/.bash_profile
		```  
3. **Create virtualEnv for the project**. 
	
	```
	mkproject -p `which python3` blockcerts
	workon blockcerts
	```	
4. **Clone git repository**  
	*If you already install gitcore, just git clone the blockcerts repo, otherwise go to the repository and download it*  
	* Download **issuer-tools** into **~/Projects/blockcerts/** folder and unzip it; next we will work on issuer-tools. 

## Create certificate template and instantiate batch


	workon blockcerts
	cd cert-tools 
	
Then create a file named *conf.ini*, and paste the following content into the file
	

	issuer_url = https://www.cosmosafe.com
	issuer_email = hello@cosmosafe.com
	issuer_name = CosmoSafe Beauty
	issuer_id = https://www.example.com/issuer.json
	revocation_list=https://www.blockcerts.org/samples/2.0/revocation-list-testnet.json
	issuer_signature_lines={"fields": [{"job_title": "CEO/FOUNDER","signature_image": "images/Signature.png","name": "Name"}]}
	issuer_public_key=ecdsa-koblitz-pubkey:mgUx2UhYD9RiF41SS6CXMDnY4etervkYFm
	# certificate information
	certificate_description = This credential represents blablabla.
	certificate_title = Certificate of Accomplishment
	criteria_narrative = Narrative...
	badge_id = 359AF970-1AF2-4FE4-BD64-E54E80DAFB90
	# images
	issuer_logo_file = images/Logo.png
	cert_image_file = images/certificate-image.png
	issuer_signature_file = images/Signature.png
	###################
	## TEMPLATE DATA ##
	###################
	data_dir = sample_data
	# template output directory
	template_dir = certificate_templates
	template_file_name = test.json
	##############################
	## INSTANTIATE BATCH CONFIG ##
	##############################
	unsigned_certificates_dir = unsigned_certificates
	roster = rosters/roster_testnet.csv
	filename_format = uuid
	no_clobber = True
	###################
	## OTHER OPTIONS ##
	###################
	# can specify an array of additional global fields. For each additional field, you must indicate:
	# additional_global_fields defines the remote url to get certificate.html to be displayed on mobile app
	additional_global_fields = {"fields": [{"path": "$.displayHtml","value": "<h1>Display your html</h1>"}, {"path": "$.@context","value": ["https://w3id.org/openbadges/v2", "https://w3id.org/blockcerts/v2", {"displayHtml": { "@id": "schema:description" }}]}]}

	
1. Line 7, add your own BTC address  
		**issuer_public_key=ecdsa-koblitz-pubkey:** following is your address.  
		
2. Line 28 is the roster of recipients you are to issue certificate to. Make sure there exist only one recepient.
	
	
3. Create *issuer.json* and upload it to the website
	
	```
	workon blockcerts
	cd cert-tools
	python cert_tools/create-v2-issuer.py -c conf.ini -o issuer.json
	```  
	
	Then host issuer.json file to public, and get the url of the **issuer.json**, like *https://www.example.com/issuer.json* for example. Modify **conf.ini** the 4th line, change **issuer_id** as **https://www.exapmle.com/issuer.json**. Next execute the above commands again to regenerate issuer.json file and reupload it.
	
4. install all necessary dependent libraries.  
	
		pip install .
	 
		
5. Copy all images(signature, logo) to sample_data/images
	
6. create certificate template
	
	```
	create-certificate-template -c conf.ini
	``` 	
	 
	
7. instantiate batch of certificates
	
	```
	instantiate-certificate-batch -c conf.ini
	```
	Then you'll have a unsigned certificate under sample\_data/unsigned\_certificates
	

### Issue a certificate.  

```
workon blockcerts
git clone https://github.com/blockchain-certificates/cert-issuer.git
cd cert-issuer
cp conf_template.ini conf.ini
python setup.py install
```

Modify the file conf.ini

```
issuing_address = <issuing-address>

# put your unsigned certificates here for signing. Default is <project-base>/data/unsigned_certificates
unsigned_certificates_dir=<path-to-your-unsigned-certificates>
# signed certificates are the output from the cert signing step; input to the cert issuing step. Default is <project-base>/data/signed_certificates
signed_certificates_dir=<path-to-your-signed-certificates>
# final blockchain certificates output. Default is <project-base>/data/unsigned_certificates
blockchain_certificates_dir=<path-to-your-blockchain-certificates>
# where to store intermediate files, for debugging and checkpointing. Default is <project-base>/data/work
work_dir=<path-to-your-workdir>

usb_name = </Volumes/path-to-usb/>
key_file = <file-you-saved-pk-to>

# which blockchain; bitcoin_regtest is default
chain=<bitcoin_regtest|bitcoin_testnet|bitcoin_mainnet|ethereum_ropsten|ethereum_mainnet|mockchain>

# this disables the wifi check, and should only be used recommended during testing
no_safe_mode
```

1. For security purpose, we can store our private key of bitcoin address in an external device, like usb drive. 
Line 12 is the path to your usb, and line 13 is the private key file name. For simplicity, you can also keep your private key file in your laptop, and point the directory to usb_name.

2. Line 16 is the blockchain mode, if you're testing, choose bitcoin_testnet; if you're working on real blockchain, choose bitcoin_mainnet

3. Now you can issue certificate by running the following command
	
	```
	python cert_issuer -c conf.ini
	```
	The output returns a txid(aka transaction id), you can search the transaction id in [BTC Explorer](https://live.blockcypher.com/btc-testnet/) to check if it's confirmed. Only after the transaction is confirmed, the certificate can be successfully verified.
	
	You would have a blockchain certificate under data/blockchain_certificates.
