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
        interceptor: RequestInterceptor? = nil
    ) -> DataRequest {
        let request = session.request(
            url,
            method: method,
            parameters: parameters,
            encoding: method == .get ? URLEncoding.default : JSONEncoding.default,
            headers: headers,
            interceptor: interceptor
        )
        
        // Add lifecycle monitoring if needed
        monitorRequestLifecycle(request)
        
        return request
    }
    
    func upload(
        _ url: String,
        fileURL: URL,
        headers: HTTPHeaders? = nil,
        interceptor: RequestInterceptor? = nil
    ) -> UploadRequest {
        let uploadRequest = session.upload(fileURL, to: url, headers: headers, interceptor: interceptor)
        
        // Add lifecycle monitoring if needed
        monitorRequestLifecycle(uploadRequest)
        
        return uploadRequest
    }
    
    func download(
        _ url: String,
        to destination: DownloadRequest.Destination? = nil,
        headers: HTTPHeaders? = nil,
        interceptor: RequestInterceptor? = nil
    ) -> DownloadRequest {
        let downloadRequest = session.download(url, to: destination, headers: headers, interceptor: interceptor)
        
        // Add lifecycle monitoring if needed
        monitorRequestLifecycle(downloadRequest)
        
        return downloadRequest
    }
    
    // MARK: - Private Methods
    private func monitorRequestLifecycle(_ request: Request) {
        request
            .response { response in
                print("Request completed: \(response.request?.url?.absoluteString ?? "Unknown URL")")
            }
            .responseJSON { response in
                if let error = response.error {
                    print("Request failed: \(error.localizedDescription)")
                } else {
                    print("Request succeeded with response: \(response)")
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
