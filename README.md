# postgresql_py_encrypt
/*1)      Download and install postgres on your machine.*/

/*2)      Start a local postgres service with the necessary user credentials to allow the creation of a database ClientDB.*/
CREATE DATABASE ClientDB;
/* creation of database */

CREATE USER client_user WITH ENCRYPTED PASSWORD 'clientuser@123';
/* creation of user */


GRANT ALL PRIVILEGES ON DATABASE ClientDB TO client_user;
/* granting all privileges on this database to this user */

\c clientdb
/*using this client database */

/* 3)      Create a table named client_credentials that has a: client_id (unique identifier), clientname, password, email, and created_on (datetime) fields.*/
CREATE TABLE Client_Credentials (
    Client_id Int Not Null ,
    Clientname VARCHAR(255),
    Password VARCHAR(16),
    Email VARCHAR(124),
    Created_on TimeStamp Default Now(),
	constraint Pk_client_id PRIMARY KEY (Client_id)
);
/*load table creation */


/* 4)      Import the data from the attached csv. */
\COPY client_credentials FROM 'client_data.csv' WITH (FORMAT CSV, HEADER);
/* copying 1000 entries from provided csv file, initially didn't work due to additional comma in a name & = sign as the first character in the password field corrected manually before running above command again*/
 
 /*5)      Encrypt the clientname, password, and email columns with an appropriate key, and assign them to new columns with the prefix encrypted_.*/
ALTER TABLE client_credentials
	ADD COLUMN
		Enc_Cname VARCHAR(255),
	ADD COLUMN
		Enc_Pwd VARCHAR(32),
	ADD COLUMN
		Enc_Email VARCHAR(124);
/*added columns for encrypted fields */


/* pip install psycopg2 */
/* The following code is run only once for the initial generation of encrypted values of the load table */
/* run LoadTable_Init Encrypt.py   */


/* 6)      Ensure that the final table does not include and sensitive information. It should therefore only include client_id, encrypted_clientname, encrypted_password, encrypted_email, created_on.*/		
CREATE TABLE Dim_client_credentials (
    Client_id Int Not Null ,
    Encrypted_clientname VARCHAR(255),
    Encrypted_Password VARCHAR(32),
    Encrypted_Email VARCHAR(124),
	Created_on TimeStamp Default Now(),
	constraint Pk_Dim_client_id PRIMARY KEY (Client_id)
);
/* dim table creation */

INSERT INTO Dim_client_credentials (Client_id, Encrypted_clientname, Encrypted_Password, Encrypted_Email, Created_on)
SELECT Client_id, Enc_Cname, Enc_Pwd, Enc_Email, Created_on
FROM client_credentials ;
/* copying 1000 entries from load file */

/* 7)      Write a python function that enables a user to retrieve the details of a client, given a client_id. The user would have access to the encryption key; however, you should think about the best way the user would be able to pass this key to the script.  */	
/* run select_function.py and enter client_id via command line to retrive matching encrypted entry */

/* 8)      Write a python function that allows the user to alter the field of a specific client, given their client_id. */
/* run update_function.py and enter client_id & other details via command line to update respective entry */
