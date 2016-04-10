import com.google.api.client.auth.oauth2.Credential;
import com.google.api.client.extensions.java6.auth.oauth2.AuthorizationCodeInstalledApp;
import com.google.api.client.extensions.jetty.auth.oauth2.LocalServerReceiver;
import com.google.api.client.googleapis.auth.oauth2.GoogleAuthorizationCodeFlow;
import com.google.api.client.googleapis.auth.oauth2.GoogleClientSecrets;
import com.google.api.client.googleapis.javanet.GoogleNetHttpTransport;
import com.google.api.client.http.HttpTransport;
import com.google.api.client.json.jackson2.JacksonFactory;
import com.google.api.client.json.JsonFactory;
import com.google.api.client.util.store.FileDataStoreFactory;

import com.google.api.client.http.FileContent;
import com.google.api.services.drive.model.File;
import com.google.api.services.drive.model.ParentReference;

import com.google.api.services.drive.Drive.Children;
import com.google.api.services.drive.model.ChildList;
import com.google.api.services.drive.model.ChildReference;

import com.google.api.client.http.GenericUrl;
import com.google.api.client.http.HttpResponse;

import java.io.InputStream;
import java.io.IOException;
import java.util.Arrays;

import com.google.api.services.drive.DriveScopes;
import com.google.api.services.drive.model.*;
import com.google.api.services.drive.Drive;

import java.io.IOException;
import java.io.InputStream;
import java.io.InputStreamReader;
import java.util.Arrays;
import java.util.List;

import java.text.SimpleDateFormat;
import java.sql.Timestamp;
import java.util.Date;
import java.util.StringTokenizer;

public class DriveQuickstart {
	/** Application name. */
	private static final String APPLICATION_NAME = "Drive API Java Quickstart";

	/** Directory to store user credentials for this application. */
	private static final java.io.File DATA_STORE_DIR = new java.io.File(System.getProperty("user.home"),
			".credentials/drive-java-quickstart.json");

	/** Global instance of the {@link FileDataStoreFactory}. */
	private static FileDataStoreFactory DATA_STORE_FACTORY;

	/** Global instance of the JSON factory. */
	private static final JsonFactory JSON_FACTORY = JacksonFactory.getDefaultInstance();

	/** Global instance of the HTTP transport. */
	private static HttpTransport HTTP_TRANSPORT;

	/**
	 * Global instance of the scopes required by this quickstart.
	 *
	 * If modifying these scopes, delete your previously saved credentials at
	 * ~/.credentials/drive-java-quickstart.json
	 */
	private static final List<String> SCOPES = Arrays.asList(DriveScopes.DRIVE_METADATA_READONLY,
			"https://www.googleapis.com/auth/drive", "https://www.googleapis.com/auth/drive.file");

	static {
		try {
			HTTP_TRANSPORT = GoogleNetHttpTransport.newTrustedTransport();
			DATA_STORE_FACTORY = new FileDataStoreFactory(DATA_STORE_DIR);
		} catch (Throwable t) {
			t.printStackTrace();
			System.exit(1);
		}
	}

	/**
	 * Creates an authorized Credential object.
	 * 
	 * @return an authorized Credential object.
	 * @throws IOException
	 */
	public static Credential authorize() throws IOException {
		// Load client secrets.
		InputStream in = DriveQuickstart.class.getResourceAsStream("/client_secret.json");
		GoogleClientSecrets clientSecrets = GoogleClientSecrets.load(JSON_FACTORY, new InputStreamReader(in));

		// Build flow and trigger user authorization request.
		GoogleAuthorizationCodeFlow flow = new GoogleAuthorizationCodeFlow.Builder(HTTP_TRANSPORT, JSON_FACTORY,
				clientSecrets, SCOPES).setDataStoreFactory(DATA_STORE_FACTORY).setAccessType("offline").build();
		Credential credential = new AuthorizationCodeInstalledApp(flow, new LocalServerReceiver()).authorize("user");
		System.out.println("Credentials saved to " + DATA_STORE_DIR.getAbsolutePath());
		return credential;
	}

	/**
	 * Build and return an authorized Drive client service.
	 * 
	 * @return an authorized Drive client service
	 * @throws IOException
	 */
	public static Drive getDriveService() throws IOException {
		Credential credential = authorize();
		return new Drive.Builder(HTTP_TRANSPORT, JSON_FACTORY, credential).setApplicationName(APPLICATION_NAME).build();
	}

	public static void main(String[] args) throws IOException {
		Drive service = getDriveService();
		File parentfolder = initializeDrive(service);
		// printFilesInFolder(service,parentfolder.getId());
		File file = insertFile(service, getSystemTimeInBelowFormat(), "Trial File", parentfolder.getId(), "",
				"/home/asifsheikh/cat.jpeg");
		System.out.println("Image successfully uploaded on drive");
	}

	private static File initializeDrive(Drive service) throws IOException {
		File file = checkFolderPresent(service);
		if (file == null) {
			File body = new File();
			body.setTitle("SmartDoorBell");
			body.setMimeType("application/vnd.google-apps.folder");
			try {
				file = service.files().insert(body).execute();
			} catch (IOException e) {
				System.out.println("An error occured: " + e);
			}
		}
		System.out.println("\n Google drive initialized");
		return file;
	}

	private static File checkFolderPresent(Drive service) throws IOException {
		FileList result = service.files().list().execute();
		List<File> files = result.getItems();
		if (files == null || files.size() == 0) {
			System.out.println("No files found.");
		} else {
			System.out.println("Files:");
			for (File file1 : files) {
				// System.out.println("The tile of file is : " +
				// file1.getTitle() + " " +
				// (file1.getTitle()).equals("SmartDoorBell"));
				if ((file1.getTitle()).equals("SmartDoorBell")) {
					return file1;
				}
			}
		}
		return null;
	}

	/**
	 * Insert new file.
	 *
	 * @param service
	 *            Drive API service instance.
	 * @param title
	 *            Title of the file to insert, including the extension.
	 * @param description
	 *            Description of the file to insert.
	 * @param parentId
	 *            Optional parent folder's ID.
	 * @param mimeType
	 *            MIME type of the file to insert.
	 * @param filename
	 *            Filename of the file to insert.
	 * @return Inserted file metadata if successful, {@code null} otherwise.
	 */
	private static File insertFile(Drive service, String title, String description, String parentId, String mimeType,
			String filename) throws IOException {
		File filetoinsert = null;
		String folderId = null;
		File body = new File();
		body.setTitle(title);
		body.setDescription(description);
		body.setMimeType(mimeType);
		filetoinsert = createDirectoryStructure(service, parentId, title);
		folderId = filetoinsert.getId();
		// Set the parent folder.
		if (folderId != null && folderId.length() > 0) {
			body.setParents(Arrays.asList(new ParentReference().setId(folderId)));
		}
		// File's content.
		java.io.File fileContent = new java.io.File(filename);
		FileContent mediaContent = new FileContent(mimeType, fileContent);
		try {
			File file = service.files().insert(body, mediaContent).execute();
			return file;
		} catch (IOException e) {
			System.out.println("An error occured: " + e);
			return null;
		}

	}

	private static File createDirectoryStructure(Drive service, String parentfolderid, String timestamp)
			throws IOException {
		File file;
		file = checkAndCreateYear(service, parentfolderid, timestamp);
		file = checkAndCreateMonth(service, file.getId(), timestamp);
		file = checkAndCreateDate(service, file.getId(), timestamp);
		return file;
	}

	private static File checkAndCreateYear(Drive service, String parentfolderid, String timestamp) throws IOException {
		// System.out.println(getYear(timestamp));
		File file = checkFilesInFolder(service, parentfolderid, getYear(timestamp));
		if (file == null) {
			File body = new File();
			body.setTitle(getYear(timestamp));
			body.setMimeType("application/vnd.google-apps.folder");
			body.setParents(Arrays.asList(new ParentReference().setId(parentfolderid)));
			try {
				file = service.files().insert(body).execute();
			} catch (IOException e) {
				System.out.println("An error occured: " + e);
			}
		}
		return file;
	}

	private static File checkAndCreateMonth(Drive service, String parentfolderid, String timestamp) throws IOException {
		// System.out.println(getMonth(timestamp));
		File file = checkFilesInFolder(service, parentfolderid, getMonth(timestamp));
		if (file == null) {
			File body = new File();
			body.setTitle(getMonth(timestamp));
			body.setMimeType("application/vnd.google-apps.folder");
			body.setParents(Arrays.asList(new ParentReference().setId(parentfolderid)));
			try {
				file = service.files().insert(body).execute();
			} catch (IOException e) {
				System.out.println("An error occured: " + e);
			}
		}
		return file;
	}

	private static File checkAndCreateDate(Drive service, String parentfolderid, String timestamp) throws IOException {
		// System.out.println(getDate(timestamp));
		File file = checkFilesInFolder(service, parentfolderid, getDate(timestamp));
		if (file == null) {
			File body = new File();
			body.setTitle(getDate(timestamp));
			body.setMimeType("application/vnd.google-apps.folder");
			body.setParents(Arrays.asList(new ParentReference().setId(parentfolderid)));
			try {
				file = service.files().insert(body).execute();
			} catch (IOException e) {
				System.out.println("An error occured: " + e);
			}
		}
		return file;
	}

	private static String getTimeStamp(String timestamp) {
		// String lines[] = String.split("\\r?\\n");
		String[] args = timestamp.split(" ");
		return args[0];
	}

	private static String getYear(String timestamp) {
		String value = getTimeStamp(timestamp);
		String[] args = value.split("-");
		return args[0];
	}

	private static String getMonth(String timestamp) {
		String value = getTimeStamp(timestamp);
		String[] args = value.split("-");
		return args[1];
	}

	private static String getDate(String timestamp) {
		String value = getTimeStamp(timestamp);
		String[] args = value.split("-");
		return args[2];
	}

	/**
	 * Insert a file into a folder.
	 *
	 * @param service
	 *            Drive API service instance.
	 * @param folderId
	 *            ID of the folder to insert the file into
	 * @param fileId
	 *            ID of the file to insert.
	 * @return The inserted parent if successful, {@code null} otherwise.
	 */
	private static ParentReference insertFileIntoFolder(Drive service, String folderId, String fileId) {
		ParentReference newParent = new ParentReference();
		newParent.setId(folderId);
		try {
			return service.parents().insert(fileId, newParent).execute();
		} catch (IOException e) {
			System.out.println("An error occurred: " + e);
		}
		return null;
	}

	/**
	 * retrive a file's metadata.
	 *
	 * @param service
	 *            Drive API service instance.
	 * @param fileId
	 *            ID of the file to print metadata for.
	 */
	private static File retriveFile(Drive service, String fileId) {
		File file = null;
		try {
			file = service.files().get(fileId).execute();
			System.out.println("Title: " + file.getTitle());
			// System.out.println("Description: " + file.getDescription());
			// System.out.println("MIME type: " + file.getMimeType());
		} catch (IOException e) {
			System.out.println("An error occured: " + e);
		}
		return file;
	}

	/**
	 * Download a file's content.
	 *
	 * @param service
	 *            Drive API service instance.
	 * @param file
	 *            Drive File instance.
	 * @return InputStream containing the file's content if successful,
	 *         {@code null} otherwise.
	 */
	private static InputStream downloadFile(Drive service, File file) {
		if (file.getDownloadUrl() != null && file.getDownloadUrl().length() > 0) {
			try {
				HttpResponse resp = service.getRequestFactory().buildGetRequest(new GenericUrl(file.getDownloadUrl()))
						.execute();
				return resp.getContent();
			} catch (IOException e) {
				// An error occurred.
				e.printStackTrace();
				return null;
			}
		} else {
			// The file doesn't have any content stored on Drive.
			return null;
		}
	}

	/**
	 * Print files belonging to a folder.
	 *
	 * @param service
	 *            Drive API service instance.
	 * @param folderId
	 *            ID of the folder to print files from.
	 */
	private static File checkFilesInFolder(Drive service, String folderId, String filetocheck) throws IOException {
		Children.List request = service.children().list(folderId);
		File file = null;
		do {
			try {
				ChildList children = request.execute();

				for (ChildReference child : children.getItems()) {
					// System.out.println("File Id: " + child.getId());
					file = retriveFile(service, child.getId());
					if ((file.getTitle()).equals(filetocheck)) {
						return file;
					}
				}
				request.setPageToken(children.getNextPageToken());
			} catch (IOException e) {
				System.out.println("An error occurred: " + e);
				request.setPageToken(null);
			}
		} while (request.getPageToken() != null && request.getPageToken().length() > 0);
		return null;
	}

	private static String getSystemTimeInBelowFormat() {
		// System.out.println(new Timestamp(date.getTime()));
		String timestamp = new SimpleDateFormat("yyyy-MMM-dd 'T' HH:mm:SS").format(new Date());
		return timestamp;
	}

}
