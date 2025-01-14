# Codebase
```swift
import Alamofire

class HTTPClient {
    // MARK: - Singleton Instance
    static let shared = HTTPClient()
    
    // MARK: - Private Properties
    private let session: Session
    private let sessionDelegate: SessionDelegate

    // MARK: - Initializer
    private init() {
        // Create a custom URLSessionConfiguration
        let configuration = URLSessionConfiguration.default
        configuration.requestCachePolicy = .reloadIgnoringLocalCacheData
        configuration.urlCache = nil // Disable URL caching
        configuration.httpCookieStorage = nil // Disable cookie storage

        // Create a custom delegate
        sessionDelegate = SessionDelegate()
        
        // Initialize the Alamofire Session with the custom configuration and delegate
        session = Session(configuration: configuration, delegate: sessionDelegate)
    }
    
    // MARK: - Public Methods
    func request(
        _ url: String,
        method: HTTPMethod = .get,
        parameters: Parameters? = nil,
        headers: HTTPHeaders? = nil,
        interceptor: RequestInterceptor? = nil,
        onProgress: ((Double) -> Void)? = nil, // Progress handler
        onComplete: @escaping (Result<Data?, AFError>) -> Void // Completion handler
    ) {
        let request = session.request(
            url,
            method: method,
            parameters: parameters,
            encoding: method == .get ? URLEncoding.default : JSONEncoding.default,
            headers: headers,
            interceptor: interceptor
        )
        
        // Monitor progress and completion
        request.downloadProgress { progress in
            onProgress?(progress.fractionCompleted)
        }
        .response { response in
            if let error = response.error {
                onComplete(.failure(error))
            } else {
                onComplete(.success(response.data))
            }
        }
    }
    
    func upload(
        _ url: String,
        fileURL: URL,
        headers: HTTPHeaders? = nil,
        interceptor: RequestInterceptor? = nil,
        onProgress: ((Double) -> Void)? = nil, // Progress handler
        onComplete: @escaping (Result<Data?, AFError>) -> Void // Completion handler
    ) {
        let uploadRequest = session.upload(fileURL, to: url, headers: headers, interceptor: interceptor)
        
        // Monitor progress and completion
        uploadRequest.uploadProgress { progress in
            onProgress?(progress.fractionCompleted)
        }
        .response { response in
            if let error = response.error {
                onComplete(.failure(error))
            } else {
                onComplete(.success(response.data))
            }
        }
    }
    
    func download(
        _ url: String,
        to destination: DownloadRequest.Destination? = nil,
        headers: HTTPHeaders? = nil,
        interceptor: RequestInterceptor? = nil,
        onProgress: ((Double) -> Void)? = nil, // Progress handler
        onComplete: @escaping (Result<URL?, AFError>) -> Void // Completion handler
    ) {
        let downloadRequest = session.download(url, to: destination, headers: headers, interceptor: interceptor)
        
        // Monitor progress and completion
        downloadRequest.downloadProgress { progress in
            onProgress?(progress.fractionCompleted)
        }
        .responseURL { response in
            if let error = response.error {
                onComplete(.failure(error))
            } else {
                onComplete(.success(response.fileURL))
            }
        }
    }
}

// MARK: - Custom URLSessionDelegate
class SessionDelegate: Alamofire.SessionDelegate {
    // Customize the delegate methods if necessary
    override func urlSession(
        _ session: URLSession,
        didBecomeInvalidWithError error: Error?
    ) {
        print("Session became invalid: \(error?.localizedDescription ?? "No error")")
    }
    
    override func urlSession(
        _ session: URLSession,
        task: URLSessionTask,
        didCompleteWithError error: Error?
    ) {
        if let error = error {
            print("Task completed with error: \(error.localizedDescription)")
        } else {
            print("Task completed successfully")
        }
    }
    
    override func urlSession(
        _ session: URLSession,
        task: URLSessionTask,
        didReceive challenge: URLAuthenticationChallenge,
        completionHandler: @escaping (URLSession.AuthChallengeDisposition, URLCredential?) -> Void
    ) {
        print("Received authentication challenge for task: \(task)")
        completionHandler(.performDefaultHandling, nil)
    }
}

