# SQLite Database Encryption using the @capacitor-community/sqlite plugin
---

*last updated on November 6, 2023 by Quéau Jean Pierre*

In that tutorial we will learned how to `encrypt` existing or new SQLite databases using the @capacitor-community/sqlite plugin.

The process flow described here does not depend on the `framework` you are using to develop your application.

The @capacitor-community/sqlite plugin enables database encryption for applications that run natively on iOS, Android, and Electron devices.

##  Table of Contents

---
 - [Process Flow](#process-flow)
    - [IsEncryption](#isencryption)
    - [Set Encryption Passphrase](#set-encryption-passphrase)
    - [OpenDatabase Method in sqliteService](#opendatabase-method-in-sqliteservice)
    - [Your Database Service](#your-database-service)
 - [Others Methods When Necessary](#others-methods-when-necessary)
    - [Is Secret/Passphrase Stored (Required)](#is-secretpassphrase-stored-required)
    - [Is Passphrase Valid (Optional)](#is-passphrase-valid-optional)
    - [Change Encryption Secret (Optional)](#change-encryption-secret-optional)
    - [Clear Encryption Passphrase (Optional)](#clear-encryption-passphrase-optional)
    - [Decrypt All Databases (Optional)](#decrypt-all-databases-optional)
    - [Get Database List (Optional)](#get-database-list-optional)
    - [Is Database Encrypted](#is-database-encrypted)

  - [Conclusion](#conclusion)

---

## Process Flow

### IsEncryption

 - Set the deviceIsEncryption to true in capacitor.config.ts file.
    ```ts
    const config: CapacitorConfig = {
    ...
        plugins: {
            CapacitorSQLite: {
                ...
                iosIsEncryption: true,
                ...
                androidIsEncryption: true,
                ...
                electronIsEncryption: true,
                ...
            }
        }
    ...
    }
    ```

 - Add `isDeviceEncryption` method in the `sqliteService.ts` file:

    ```ts 
    export interface ISQLiteService {
        ...
        isDeviceEncryption(): Promise<boolean>
    };    
    ```

    ```ts 
    class SQLiteService implements ISQLiteService {
        platform = Capacitor.getPlatform();
        sqlitePlugin = CapacitorSQLite;
        sqliteConnection = new SQLiteConnection(CapacitorSQLite);
        dbNameVersionDict: Map<string, number> = new Map();

        ... 

        async isDeviceEncryption(): Promise<boolean> { 
            // Is platform allow for encryption
            const isPlatformEncryption = this.platform === 'web' ? false : true;
            // Check if isEncryption in the capacitor.config.ts
            const isEncryptInConfig = (await this.sqliteConnection.isInConfigEncryption()).result

            return isPlatformEncryption && isEncryptInConfig ? true : false;
        }
    }
    ```  

### Set Encryption Passphrase

 - This methods allows to set the `passphrase` which is required to encrypt databases.

 - Add `setEncryptionPassphrase` method in the `sqliteService.ts` file:

    ```ts 
    export interface ISQLiteService {
        ...
        setEncryptionPassphrase(passphrase: string): Promise<void>
    };    
    ```

    ```ts 
    class SQLiteService implements ISQLiteService {
        platform = Capacitor.getPlatform();
        sqlitePlugin = CapacitorSQLite;
        sqliteConnection = new SQLiteConnection(CapacitorSQLite);
        dbNameVersionDict: Map<string, number> = new Map();

        ... 

        async setEncryptionPassphrase(passphrase: string): Promise<void> {
            // check if a passphrase is already stored
            let isSetPassphrase = await this.isSecretStored();
            if(isSetPassphrase) {
                const msg = "Passphrase already stored";
                throw new Error(`sqliteService.setEncryptionPassphrase: ${msg}`);
            }
            return Promise.resolve(await this.sqliteConnection.setEncryptionSecret(passphrase));
        }
        ... 
    }
    ```  

### OpenDatabase Method in sqliteService

 - This method will allow to: 
    - create/open a non-encrypted database
    - create/open an encrypted database
    - open a non-encrypted database and encrypt it
    - open an encrypted database and decrypt it

 - create or modify `openDatabase` method in the `sqliteService.ts` file as followed: 
     ```ts 
    export interface ISQLiteService {
        ...
    openDatabase(dbName:string, version: number,
                 readOnly: boolean, decrypt?: boolean)
                : Promise<SQLiteDBConnection>;
    };    
    ```

    ```ts 
    class SQLiteService implements ISQLiteService {
        platform = Capacitor.getPlatform();
        sqlitePlugin = CapacitorSQLite;
        sqliteConnection = new SQLiteConnection(CapacitorSQLite);
        dbNameVersionDict: Map<string, number> = new Map();

        ... 

    async openDatabase(dbName:string, version: number,
                       readOnly: boolean,
                       decrypt: boolean = false)
                : Promise<SQLiteDBConnection>  {
        this.dbNameVersionDict.set(dbName, version);
        let db: SQLiteDBConnection;
        const retCC = (await this.sqliteConnection.checkConnectionsConsistency()).result;
        let isConn = (await this.sqliteConnection.isConnection(dbName, readOnly)).result;
        if(retCC && isConn) {
            db = await this.sqliteConnection.retrieveConnection(dbName, readOnly);
        } else {
            let encrypted = false;
            let mode = "no-encryption";
            if(this.platform !== "web") {
                const isDbExists = (await this.sqliteConnection.isDatabase(dbName)).result!;
                const isDevEncrypt = await this.isDeviceEncryption();
                const isDbEncrypt = isDevEncrypt && isDbExists
                        ? (await this.sqliteConnection.isDatabaseEncrypted(dbName)).result!
                        : false;
                const isPassphrase = isDevEncrypt 
                        ? await this.isSecretStored()
                        : false;
                encrypted = (!isDbExists && isDevEncrypt && isPassphrase) ||
                             (isDbExists && isDevEncrypt && isDbEncrypt && isPassphrase) ||
                             (isDbExists && isDevEncrypt && !isDbEncrypt && isPassphrase ) ? true : false;
                mode =  isDbExists && isDevEncrypt && !isDbEncrypt && isPassphrase
                            ? "encryption" 
                            : encrypted && decrypt ? "decryption"
                            : encrypted ? "secret" : "no-encryption";
            }
            db = await this.sqliteConnection
                    .createConnection(dbName, encrypted, mode, version, readOnly);
        }
    
        await db.open();
        const res = (await db.isDBOpen()).result;
        if(res) {
            return db;
        } else {
            const msg = "database not opened";
            throw new Error(`sqliteService.openDatabase: ${msg}`);
        }

    }
    ```  

### Your Database Service

 - In `initializeDatabase` method or whatever you have call them, replace the call of sqliteServ.openDatabase with:

    ```ts
    this.db = await this.sqliteServ.openDatabase(this.database,
                                                 this.loadToVersion,
                                                 false);
    ```

    here understand `this.database` as the database name and `this.loadToVersion` as the database version.



## Others Methods When Necessary

### Is Secret/Passphrase Stored (Required)

 - This method allows to know if a `passphrase` is already stored for your application.

 - Add `isSecretStored` method in the `sqliteService.ts` file:

    ```ts 
    export interface ISQLiteService {
        ...
        isSecretStored(): Promise<boolean>;
    };    
    ```

    ```ts 
    class SQLiteService implements ISQLiteService {
        platform = Capacitor.getPlatform();
        sqlitePlugin = CapacitorSQLite;
        sqliteConnection = new SQLiteConnection(CapacitorSQLite);
        dbNameVersionDict: Map<string, number> = new Map();

        ... 

        async isSecretStored(): Promise<boolean> {
            const res = await this.sqliteConnection.isSecretStored()
            return  res.result ?? false;
        }

        ... 

    }
    ```

### Is Passphrase Valid (Optional)

 - This method allow to check if a given passphrase is equal to the one stored

 - Add `isPassphraseValid` method in the `sqliteService.ts` file:

    ```ts 
    export interface ISQLiteService {
        ...
        isPassphraseValid(passphrase: string): Promise<boolean>;
    };    
    ```

    ```ts 
    class SQLiteService implements ISQLiteService {
        platform = Capacitor.getPlatform();
        sqlitePlugin = CapacitorSQLite;
        sqliteConnection = new SQLiteConnection(CapacitorSQLite);
        dbNameVersionDict: Map<string, number> = new Map();

        ... 

        async isPassphraseValid(passphrase: string): Promise<boolean> {
            const res = await this.sqliteConnection.checkEncryptionSecret(passphrase);
            return  res.result ?? false;
        }

        ... 

    }
    ```

### Change Encryption Secret (Optional)

 - This method allow to change the stored passphrase with a new one

 - Add `changeEncryptionSecret` method in the `sqliteService.ts` file:

    ```ts 
    export interface ISQLiteService {
        ...
        changeEncryptionSecret(passphrase: string, oldpassphrase: string)
                                                        : Promise<void>
    };    
    ```

    ```ts 
    class SQLiteService implements ISQLiteService {
        platform = Capacitor.getPlatform();
        sqlitePlugin = CapacitorSQLite;
        sqliteConnection = new SQLiteConnection(CapacitorSQLite);
        dbNameVersionDict: Map<string, number> = new Map();

        ... 

        async changeEncryptionSecret(passphrase: string,
                                     oldpassphrase: string)
                                     : Promise<void> {
            return await this.sqliteConnection.changeEncryptionSecret(
                                            passphrase, oldpassphrase);
        }
    
        ... 

    }
    ```

### Clear Encryption Passphrase (Optional)

 - This method allow to clear the stored passphrase 

 - Add `clearEncryptionPassphrase` method in the `sqliteService.ts` file:

    ```ts 
    export interface ISQLiteService {
        ...
        clearEncryptionPassphrase(): Promise<void>;
    };    
    ```

    ```ts 
    class SQLiteService implements ISQLiteService {
        platform = Capacitor.getPlatform();
        sqlitePlugin = CapacitorSQLite;
        sqliteConnection = new SQLiteConnection(CapacitorSQLite);
        dbNameVersionDict: Map<string, number> = new Map();

        ... 

        async clearEncryptionPassphrase(): Promise<void> {
            // close all open connections
            await this.sqliteConnection.closeAllConnections();
            return await this.sqliteConnection.clearEncryptionSecret();
        }
   
        ... 

    }
    ```

### Decrypt All Databases (Optional)

 - This method allow to decrypt all databases in the database folder 

 - Add `decryptAllDatabases` method in the `sqliteService.ts` file:

    ```ts 
    export interface ISQLiteService {
        ...
        decryptAllDatabases(dbVerDict: Map<string, number>): Promise<void>;
    };    
    ```

    ```ts 
    class SQLiteService implements ISQLiteService {
        platform = Capacitor.getPlatform();
        sqlitePlugin = CapacitorSQLite;
        sqliteConnection = new SQLiteConnection(CapacitorSQLite);
        dbNameVersionDict: Map<string, number> = new Map();

        ... 

        async decryptAllDatabases(dbVerDict: Map<string, number>): Promise<void> {

            // close all open connections
            await this.sqliteConnection.closeAllConnections();
            // get the database list
            const dbList = await this.getDatabaseList();
            const encryptNames = dbList.filter(item => item.encrypted === true).map(item => item.name);
            for (const name of encryptNames) {
                const version = dbVerDict.get(name) ?? 1;
                // decrypt the database
                await this.openDatabase(name, version, false, true);
            }
            // close all open connections
            await this.sqliteConnection.closeAllConnections();
            // clear the passphrase
            await this.sqliteConnection.clearEncryptionSecret();
        }
    
        ... 

    }
    ```

### Get Database List (Optional)

 - This method allow to get the list of all databases in the database folder 

 - Add `getDatabaseList` method in the `sqliteService.ts` file:

    ```ts 
    export interface ISQLiteService {
        ...
        getDatabaseList(): Promise<any[]>;
    };    
    ```

    ```ts 
    class SQLiteService implements ISQLiteService {
        platform = Capacitor.getPlatform();
        sqlitePlugin = CapacitorSQLite;
        sqliteConnection = new SQLiteConnection(CapacitorSQLite);
        dbNameVersionDict: Map<string, number> = new Map();

        ... 

        async getDatabaseList(): Promise<any[]> {
            const retDbList: TDatabase[] = [];
            const dbList = (await this.sqliteConnection.getDatabaseList()).values!;
            if(dbList.length > 0) {
                for (let idx:number = 0; idx < dbList.length; idx++) {
                    const dbName = dbList[idx].split("SQLite.db")[0];
                    const isEncrypt = (await this.sqliteConnection.isDatabaseEncrypted(dbName)).result!;
                    const data: TDatabase = {name: dbName, encrypted: isEncrypt};
                    retDbList.push(data);
                }
            }
            return retDbList;
        };
        
        ... 

    }
    ```

### Is Database Encrypted

 - This methods allows to check if a database exists and is encrypted

 - Add `isDatabaseEncrypted` method in the `sqliteService.ts` file:

    ```ts
    async isDatabaseEncrypted(dbName: string): Promise<boolean> {
        if((await this.sqliteConnection.isDatabase(dbName)).result)  {
            const isDBEncrypted = (await this.sqliteConnection.isDatabaseEncrypted(dbName)).result;
            return isDBEncrypted!
        } else {
            const msg = `database ${dbName} does not exist` ;
            throw new Error(`sqliteService.isDatabaseEncrypted: ${msg}`);
        }       
    }
    ```

## Conclusion 

We have completed SQLite Database Encryption using the `@capacitor-community/sqlite plugin`.

We learned about the process flow required to encrypt the database
 - by first modifying the `capacitor.config.ts` file, 
 - set the encryption passphrase in the sqliteService
 - then how to open the database in the sqliteService
 - call the sqliteService openDatabase from your own service or compon
ent.

We learned how to enhance the sqliteService with some methods that you may need during your application Development.

All this learning is not Framework dependent

An example for the Vue Framework can be found  at [Encryption/ionic7-vue-sqlite-app-encryption](https://github.com/jepiqueau/blog-tutorials-apps/tree/main/SQLite/Encryption/ionic7-vue-sqlite-app-encryption)



 
