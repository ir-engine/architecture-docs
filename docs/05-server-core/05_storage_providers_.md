# Storage providers

## Overview

The Storage Providers component is an essential element of the iR Engine's server core that manages the storage and retrieval of files such as images, videos, and 3D models. It provides a unified interface for file operations while abstracting the underlying storage implementation, whether local filesystem, Amazon S3, or Google Cloud Storage. 

By implementing this abstraction layer, the system enables flexibility in storage solutions without requiring changes to application code. This chapter explores the implementation, interface, and usage of storage providers within the iR Engine.

## Core concepts

### Storage abstraction

The storage abstraction provides a consistent interface for file operations:

- **Implementation independence**: Separates file operation logic from storage implementation details
- **Unified API**: Provides a standard set of methods for all storage backends
- **Interchangeability**: Allows switching between storage solutions with minimal code changes
- **Configuration-driven**: Uses application settings to determine the active storage provider
- **Extensibility**: Supports adding new storage backends without changing application code

This abstraction creates a flexible foundation for file management.

### Provider implementations

The system includes multiple storage provider implementations:

- **Local storage**: Stores files on the server's filesystem
- **S3 storage**: Stores files in Amazon S3 buckets
- **GCS storage**: Stores files in Google Cloud Storage
- **Custom providers**: Can be added to support additional storage solutions

Each implementation handles the specifics of its storage backend while conforming to the common interface.

### File operations

Storage providers support a standard set of file operations:

- **Upload**: Store files in the selected storage system
- **Download**: Retrieve files from storage
- **URL generation**: Create URLs for accessing stored files
- **Listing**: Enumerate files in a directory or folder
- **Deletion**: Remove files from storage
- **Existence checking**: Verify if a file exists
- **Directory operations**: Create, check, and manage directories

These operations cover the essential needs for file management in applications.

## Implementation

### Provider interface

The storage provider interface defines the contract for all implementations:

```typescript
// Simplified from: src/media/storageprovider/storageprovider.interface.ts
import { PassThrough } from 'stream';

/**
 * Interface for object upload parameters
 */
export interface StorageObjectPutInterface {
  // The file's path/name in storage
  Key: string;
  
  // The file content (buffer or stream)
  Body: Buffer | PassThrough;
  
  // The MIME type of the file
  ContentType: string;
  
  // Optional metadata
  Metadata?: Record<string, string>;
  
  // Other optional parameters
  ContentEncoding?: string;
  CacheControl?: string;
}

/**
 * Interface that all storage providers must implement
 */
export interface StorageProviderInterface {
  /**
   * Uploads a file to storage
   * @param object Upload parameters
   * @param params Additional provider-specific parameters
   * @returns Promise resolving to success status
   */
  putObject(object: StorageObjectPutInterface, params?: any): Promise<boolean>;
  
  /**
   * Retrieves a file from storage
   * @param key File path/name
   * @returns Promise resolving to file content and metadata
   */
  getObject(key: string): Promise<{ Body: Buffer, ContentType: string }>;
  
  /**
   * Gets a URL for accessing a file
   * @param key File path/name
   * @param internal Whether the URL is for internal use
   * @returns URL string
   */
  getCachedURL(key: string, internal?: boolean): string;
  
  /**
   * Lists files in a folder
   * @param folderName Folder path
   * @param recursive Whether to include subfolders
   * @returns Promise resolving to file information
   */
  listFolderContent(folderName: string, recursive?: boolean): Promise<any[]>;
  
  /**
   * Deletes files from storage
   * @param keys Array of file paths/names
   * @returns Promise resolving to deletion result
   */
  deleteResources(keys: string[]): Promise<any>;
  
  /**
   * Checks if a file exists
   * @param fileName File name
   * @param directoryPath Directory path
   * @returns Promise resolving to existence status
   */
  doesExist(fileName: string, directoryPath: string): Promise<boolean>;
  
  /**
   * Checks if a path is a directory
   * @param fileName File name
   * @param directoryPath Directory path
   * @returns Promise resolving to directory status
   */
  isDirectory(fileName: string, directoryPath: string): Promise<boolean>;
  
  // Additional methods like moveObject, getSignedUrl, etc.
}
```

This interface:
- Defines the contract that all storage providers must implement
- Specifies the parameters and return types for each method
- Provides a consistent API for file operations
- Enables interchangeability between different storage implementations

### Local storage implementation

The local storage provider stores files on the server's filesystem:

```typescript
// Simplified from: src/media/storageprovider/local.storage.ts
import fs from 'fs';
import path from 'path';
import { StorageProviderInterface, StorageObjectPutInterface } from './storageprovider.interface';
import config from '../../appconfig';

/**
 * Storage provider that uses the local filesystem
 */
export class LocalStorage implements StorageProviderInterface {
  // Base directory for file storage
  PATH_PREFIX: string;
  
  /**
   * Constructor
   */
  constructor() {
    // Set up the base directory from configuration
    this.PATH_PREFIX = path.join(process.cwd(), config.storage.local.baseDir || 'upload');
    
    // Create the directory if it doesn't exist
    if (!fs.existsSync(this.PATH_PREFIX)) {
      fs.mkdirSync(this.PATH_PREFIX, { recursive: true });
    }
  }
  
  /**
   * Uploads a file to the local filesystem
   * @param data Upload parameters
   * @returns Promise resolving to success status
   */
  async putObject(data: StorageObjectPutInterface): Promise<boolean> {
    // Construct the full file path
    const filePath = path.join(this.PATH_PREFIX, data.Key);
    
    // Ensure the directory exists
    const directoryPath = path.dirname(filePath);
    if (!fs.existsSync(directoryPath)) {
      fs.mkdirSync(directoryPath, { recursive: true });
    }
    
    // Write the file
    if (Buffer.isBuffer(data.Body)) {
      // Handle Buffer data
      fs.writeFileSync(filePath, data.Body);
    } else {
      // Handle stream data
      return new Promise((resolve, reject) => {
        const writeStream = fs.createWriteStream(filePath);
        data.Body.pipe(writeStream);
        
        writeStream.on('finish', () => resolve(true));
        writeStream.on('error', (err) => reject(err));
      });
    }
    
    return true;
  }
  
  /**
   * Gets a URL for accessing a file
   * @param key File path/name
   * @returns URL string
   */
  getCachedURL(key: string): string {
    // For local storage, construct a URL to the local server
    const domain = config.storage.local.domain || 'localhost:3030';
    const protocol = config.storage.local.protocol || 'http';
    
    return `${protocol}://${domain}/uploads/${key}`;
  }
  
  /**
   * Lists files in a folder
   * @param folderName Folder path
   * @param recursive Whether to include subfolders
   * @returns Promise resolving to file information
   */
  async listFolderContent(folderName: string, recursive = false): Promise<any[]> {
    const folderPath = path.join(this.PATH_PREFIX, folderName);
    
    // Check if the folder exists
    if (!fs.existsSync(folderPath)) {
      return [];
    }
    
    // Read the directory
    const entries = fs.readdirSync(folderPath, { withFileTypes: true });
    
    // Process each entry
    const results = [];
    for (const entry of entries) {
      const entryPath = path.join(folderName, entry.name);
      
      if (entry.isDirectory()) {
        // Handle directory
        results.push({
          key: entryPath + '/',
          name: entry.name,
          type: 'folder',
          size: 0
        });
        
        // Include subdirectory contents if recursive
        if (recursive) {
          const subResults = await this.listFolderContent(entryPath, true);
          results.push(...subResults);
        }
      } else {
        // Handle file
        const stats = fs.statSync(path.join(folderPath, entry.name));
        const extension = path.extname(entry.name).substring(1);
        
        results.push({
          key: entryPath,
          name: path.basename(entry.name, '.' + extension),
          type: extension,
          size: stats.size,
          lastModified: stats.mtime
        });
      }
    }
    
    return results;
  }
  
  /**
   * Deletes files from storage
   * @param keys Array of file paths/names
   * @returns Promise resolving to deletion result
   */
  async deleteResources(keys: string[]): Promise<any> {
    for (const key of keys) {
      const filePath = path.join(this.PATH_PREFIX, key);
      
      if (fs.existsSync(filePath)) {
        const stats = fs.statSync(filePath);
        
        if (stats.isDirectory()) {
          // Remove directory recursively
          fs.rmdirSync(filePath, { recursive: true });
        } else {
          // Remove file
          fs.unlinkSync(filePath);
        }
      }
    }
    
    return { deleted: keys };
  }
  
  // Implementations for other interface methods
}
```

This implementation:
- Uses the Node.js filesystem API to store and manage files
- Constructs file paths based on the configured base directory
- Handles both buffer and stream data for file uploads
- Generates URLs that point to a local file server
- Provides directory listing with file metadata
- Implements file and directory deletion

### S3 storage implementation

The S3 storage provider stores files in Amazon S3 buckets:

```typescript
// Simplified from: src/media/storageprovider/s3.storage.ts
import { S3Client, PutObjectCommand, GetObjectCommand, ListObjectsV2Command, DeleteObjectsCommand } from '@aws-sdk/client-s3';
import { StorageProviderInterface, StorageObjectPutInterface } from './storageprovider.interface';
import config from '../../appconfig';

/**
 * Storage provider that uses Amazon S3
 */
export class S3Storage implements StorageProviderInterface {
  // S3 client instance
  private s3Client: S3Client;
  
  // S3 bucket name
  private bucket: string;
  
  // CDN domain for URLs
  private cdnDomain: string;
  
  /**
   * Constructor
   */
  constructor() {
    // Create S3 client with configuration
    this.s3Client = new S3Client({
      region: config.storage.s3.region,
      credentials: {
        accessKeyId: config.storage.s3.accessKeyId,
        secretAccessKey: config.storage.s3.secretAccessKey
      }
    });
    
    // Set bucket and CDN domain
    this.bucket = config.storage.s3.bucket;
    this.cdnDomain = config.storage.s3.cdnDomain || `${this.bucket}.s3.amazonaws.com`;
  }
  
  /**
   * Uploads a file to S3
   * @param data Upload parameters
   * @returns Promise resolving to success status
   */
  async putObject(data: StorageObjectPutInterface): Promise<boolean> {
    // Create command for S3 upload
    const command = new PutObjectCommand({
      Bucket: this.bucket,
      Key: data.Key,
      Body: data.Body,
      ContentType: data.ContentType,
      Metadata: data.Metadata,
      ContentEncoding: data.ContentEncoding,
      CacheControl: data.CacheControl
    });
    
    // Execute the command
    await this.s3Client.send(command);
    
    return true;
  }
  
  /**
   * Gets a URL for accessing a file
   * @param key File path/name
   * @returns URL string
   */
  getCachedURL(key: string): string {
    // Construct URL using CDN domain
    return `https://${this.cdnDomain}/${key}`;
  }
  
  /**
   * Lists files in a folder
   * @param folderName Folder path
   * @param recursive Whether to include subfolders
   * @returns Promise resolving to file information
   */
  async listFolderContent(folderName: string, recursive = false): Promise<any[]> {
    // Ensure folder name ends with a slash
    const prefix = folderName.endsWith('/') ? folderName : folderName + '/';
    
    // Create command for listing objects
    const command = new ListObjectsV2Command({
      Bucket: this.bucket,
      Prefix: prefix,
      Delimiter: recursive ? undefined : '/'
    });
    
    // Execute the command
    const response = await this.s3Client.send(command);
    
    // Process the results
    const results = [];
    
    // Process common prefixes (folders)
    if (response.CommonPrefixes) {
      for (const prefix of response.CommonPrefixes) {
        if (prefix.Prefix) {
          results.push({
            key: prefix.Prefix,
            name: prefix.Prefix.split('/').slice(-2)[0],
            type: 'folder',
            size: 0
          });
        }
      }
    }
    
    // Process objects (files)
    if (response.Contents) {
      for (const object of response.Contents) {
        if (object.Key && object.Key !== prefix) {
          const name = object.Key.split('/').pop() || '';
          const extension = name.includes('.') ? name.split('.').pop() || '' : '';
          
          results.push({
            key: object.Key,
            name: name.replace(`.${extension}`, ''),
            type: extension,
            size: object.Size || 0,
            lastModified: object.LastModified
          });
        }
      }
    }
    
    return results;
  }
  
  /**
   * Deletes files from storage
   * @param keys Array of file paths/names
   * @returns Promise resolving to deletion result
   */
  async deleteResources(keys: string[]): Promise<any> {
    // Create command for deleting objects
    const command = new DeleteObjectsCommand({
      Bucket: this.bucket,
      Delete: {
        Objects: keys.map(key => ({ Key: key }))
      }
    });
    
    // Execute the command
    const response = await this.s3Client.send(command);
    
    return {
      deleted: response.Deleted?.map(obj => obj.Key) || []
    };
  }
  
  // Implementations for other interface methods
}
```

This implementation:
- Uses the AWS SDK to interact with Amazon S3
- Creates an S3 client configured with credentials from application settings
- Maps interface methods to S3-specific commands
- Handles S3 bucket operations for file management
- Generates URLs that point to S3 or a CDN
- Processes S3-specific responses into a consistent format

### Provider factory

The provider factory creates and manages storage provider instances:

```typescript
// Simplified from: src/media/storageprovider/storageprovider.ts
import config from '../../appconfig';
import { StorageProviderInterface } from './storageprovider.interface';
import LocalStorage from './local.storage';
import S3Storage from './s3.storage';
import GCSStorage from './gcs.storage';

// Cache of provider instances
const providers: Record<string, StorageProviderInterface> = {};

// Map of provider types to implementations
const storageImplementations: Record<string, any> = {
  local: LocalStorage,
  s3: S3Storage,
  gcs: GCSStorage
};

/**
 * Gets a storage provider instance
 * @param providerName Provider name (default: 'default')
 * @returns Storage provider instance
 */
export const getStorageProvider = (providerName = 'default'): StorageProviderInterface => {
  // Create the default provider if it doesn't exist
  if (!providers[providerName]) {
    if (providerName === 'default') {
      createDefaultStorageProvider();
    } else {
      throw new Error(`Storage provider '${providerName}' not found`);
    }
  }
  
  return providers[providerName];
};

/**
 * Creates the default storage provider
 * @returns Storage provider instance
 */
export const createDefaultStorageProvider = (): StorageProviderInterface => {
  // Get provider type from configuration
  const providerType = config.storage.provider || 'local';
  
  // Get the provider implementation class
  const ProviderClass = storageImplementations[providerType] || LocalStorage;
  
  // Create an instance
  const instance = new ProviderClass();
  
  // Store the instance
  providers['default'] = instance;
  
  return instance;
};
```

This factory:
- Maintains a cache of provider instances
- Maps provider types to implementation classes
- Creates the default provider based on configuration
- Provides a consistent way to access the active provider
- Supports multiple provider instances if needed

## Usage examples

Storage providers are used throughout the application for file operations:

### Uploading files

Files are uploaded using the `putObject` method:

```typescript
// Example of file upload
import { getStorageProvider } from '../storageprovider/storageprovider';

/**
 * Uploads a file to storage
 * @param fileBuffer File content
 * @param fileName File name
 * @param contentType MIME type
 * @param directory Directory path
 * @returns Promise resolving to the file key
 */
async function uploadFile(
  fileBuffer: Buffer,
  fileName: string,
  contentType: string,
  directory: string
): Promise<string> {
  // Get the storage provider
  const storage = getStorageProvider();
  
  // Construct the file key
  const key = `${directory}/${fileName}`;
  
  // Upload the file
  await storage.putObject({
    Key: key,
    Body: fileBuffer,
    ContentType: contentType
  });
  
  console.log(`File ${key} uploaded successfully`);
  
  return key;
}
```

This function:
1. Gets the active storage provider
2. Constructs a key (path) for the file
3. Uploads the file using the provider's `putObject` method
4. Returns the file key for future reference

### Getting file URLs

URLs for accessing files are generated using the `getCachedURL` method:

```typescript
// Example of URL generation
import { getStorageProvider } from '../storageprovider/storageprovider';

/**
 * Gets a URL for a file
 * @param key File key
 * @returns URL string
 */
function getFileUrl(key: string): string {
  // Get the storage provider
  const storage = getStorageProvider();
  
  // Get the URL
  const url = storage.getCachedURL(key);
  
  console.log(`URL for ${key}: ${url}`);
  
  return url;
}
```

This function:
1. Gets the active storage provider
2. Generates a URL for the file using the provider's `getCachedURL` method
3. Returns the URL for use in the application

### Listing files

Files in a directory are listed using the `listFolderContent` method:

```typescript
// Example of directory listing
import { getStorageProvider } from '../storageprovider/storageprovider';

/**
 * Lists files in a directory
 * @param directory Directory path
 * @param recursive Whether to include subdirectories
 * @returns Promise resolving to file information
 */
async function listFiles(directory: string, recursive = false): Promise<any[]> {
  // Get the storage provider
  const storage = getStorageProvider();
  
  // List the directory contents
  const files = await storage.listFolderContent(directory, recursive);
  
  console.log(`Found ${files.length} files in ${directory}`);
  
  return files;
}
```

This function:
1. Gets the active storage provider
2. Lists the contents of a directory using the provider's `listFolderContent` method
3. Returns the file information for use in the application

### Deleting files

Files are deleted using the `deleteResources` method:

```typescript
// Example of file deletion
import { getStorageProvider } from '../storageprovider/storageprovider';

/**
 * Deletes files from storage
 * @param keys Array of file keys
 * @returns Promise resolving to deletion result
 */
async function deleteFiles(keys: string[]): Promise<any> {
  // Get the storage provider
  const storage = getStorageProvider();
  
  // Delete the files
  const result = await storage.deleteResources(keys);
  
  console.log(`Deleted ${result.deleted.length} files`);
  
  return result;
}
```

This function:
1. Gets the active storage provider
2. Deletes the files using the provider's `deleteResources` method
3. Returns the deletion result for confirmation

## Integration with other components

The storage providers integrate with several other components of the server core:

### Application configuration

The storage provider system uses configuration values:

```typescript
// Example of configuration integration
import config from '../../appconfig';

// In createDefaultStorageProvider
const providerType = config.storage.provider || 'local';

// In LocalStorage constructor
this.PATH_PREFIX = path.join(process.cwd(), config.storage.local.baseDir || 'upload');

// In S3Storage constructor
this.s3Client = new S3Client({
  region: config.storage.s3.region,
  credentials: {
    accessKeyId: config.storage.s3.accessKeyId,
    secretAccessKey: config.storage.s3.secretAccessKey
  }
});
```

This integration:
- Uses configuration values to determine the active provider
- Applies provider-specific settings from configuration
- Enables changing storage solutions through configuration
- Supports environment-specific storage settings

### Services

Services use storage providers for file operations:

```typescript
// Example of service integration
import { getStorageProvider } from '../storageprovider/storageprovider';
import { Application } from '../../declarations';

/**
 * Service for managing file uploads
 */
export class FileUploadService {
  app: Application;
  
  constructor(options: any, app: Application) {
    this.app = app;
  }
  
  /**
   * Handles file upload
   * @param id Resource ID
   * @param data Upload data
   * @returns Promise resolving to upload result
   */
  async create(id: null, data: any): Promise<any> {
    // Get the storage provider
    const storage = getStorageProvider();
    
    // Construct the file key
    const key = `uploads/${data.directory}/${data.fileName}`;
    
    // Upload the file
    await storage.putObject({
      Key: key,
      Body: data.file,
      ContentType: data.contentType
    });
    
    // Create a database record for the file
    const record = await this.app.service('static-resource').create({
      name: data.fileName,
      key: key,
      contentType: data.contentType,
      url: storage.getCachedURL(key)
    });
    
    return record;
  }
}
```

This integration:
- Uses the storage provider for file uploads
- Creates database records for uploaded files
- Generates URLs for accessing files
- Provides a consistent API for file operations

## Benefits of storage providers

The Storage Providers component provides several key advantages:

1. **Abstraction**: Separates file operation logic from storage implementation details
2. **Flexibility**: Enables switching between storage solutions with minimal code changes
3. **Standardization**: Provides a consistent API for file operations
4. **Scalability**: Supports cloud storage solutions for handling large volumes of files
5. **Maintainability**: Isolates storage-specific code in dedicated implementations
6. **Testability**: Allows mocking storage operations for testing
7. **Extensibility**: Supports adding new storage backends as needed

These benefits make storage providers an essential component for managing files in the iR Engine's server core.

## Next steps

With an understanding of how the application manages files, the next chapter explores how to add custom logic to service operations.

Next: [Hooks (FeathersJS)](06_hooks__feathersjs__.md)

---


