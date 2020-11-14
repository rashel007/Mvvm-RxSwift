# Mvvm-RxSwift
Building a login screen using mvvm and rxswift. 

### Model

```swift
struct UserCredential {
    let userName: String
    let password: String
}
```

### ViewModel

```swift
import RxRelay
import RxSwift

class LoginViewModel {
    
    var userName = BehaviorRelay<String>(value: "")
    var password = BehaviorRelay<String>(value: "")
    var result = BehaviorRelay<String>(value: "")
    var loginButtonTapped  = PublishRelay<Void>()
    
    
    var isValid: Observable<Bool> {
        return Observable<Bool>.combineLatest(self.userName.asObservable(), self.password.asObservable()) { (_userName, _password) -> Bool in
            return !(_userName.isEmpty || _password.isEmpty)
        }
    }
    
    
    private let loginService: LoginService
    private let disposeBag = DisposeBag()
    
    init(loginService: LoginService) {
        self.loginService = loginService
        self.makeSubscriptions()
    }
    
    private func makeSubscriptions() {
    
        self.loginButtonTapped.subscribe(onNext: { [weak self] in
            guard let `self` = self else {return}
            self.attemptLogin(userName: self.userName.value, password: self.password.value)
        }).disposed(by: disposeBag)
    }
    
    private func attemptLogin(userName: String, password: String){
        let userCred = UserCredential(userName: userName, password: password)
        self.loginService.login(userCredential: userCred) { [weak self] (isValidCred) in
            
            guard let `self` = self else {return}
            
            self.result.accept(isValidCred ? "Login Successful" : "Login failed")
        }
    }
}
```

### View

```swift
import UIKit
import RxSwift
import RxCocoa

class ViewController: UIViewController {
    
    @IBOutlet weak var usernameTextField: UITextField!
    @IBOutlet weak var passwordTextField: UITextField!
    @IBOutlet weak var loginButton: UIButton!
    
    let viewModel = LoginViewModel(loginService: LoginService())
    let disposeBag = DisposeBag()

    override func viewDidLoad() {
        super.viewDidLoad()
        // Do any additional setup after loading the view.
        self.bindViews()
        self.bindState()
    }
    
    
    private func bindViews() {
        self.usernameTextField.rx.text
            .orEmpty
            .bind(to: self.viewModel.userName)
            .disposed(by: disposeBag)
        
        self.passwordTextField.rx.text
            .orEmpty
            .bind(to: self.viewModel.password)
            .disposed(by: disposeBag)
        
        self.loginButton.rx.tap
            .bind(to: self.viewModel.loginButtonTapped)
            .disposed(by: disposeBag)
    }
    
    private func bindState() {
        self.viewModel.isValid
            .subscribe(onNext: { [weak self] (isValid) in
                guard let `self` = self else { return }
                self.updateLoginButton(isValid: isValid)
            }).disposed(by: disposeBag)
        
        self.viewModel.result.asObservable()
            .subscribe(onNext: { [weak self] (message) in
                guard let `self` = self else { return }
                self.showAlert(message: message)
            }).disposed(by: disposeBag)
    }
    
    private func updateLoginButton(isValid: Bool){
        self.loginButton.backgroundColor = isValid ? .black : .gray
        self.loginButton.isEnabled = isValid
    }
    
    private func showAlert(message: String) {
        let alert = UIAlertController(title: "", message: message, preferredStyle: .alert)
        let okAction = UIAlertAction(title: "OK", style: .default, handler: nil)
        
        alert.addAction(okAction)
        
        self.present(alert, animated: true, completion: nil)
    }


}
```
