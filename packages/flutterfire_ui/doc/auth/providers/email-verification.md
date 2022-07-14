# Email verification in FlutterFire UI

FlutterFire UI provides a pre-built `EmailVerificationScreen`:

```dart
class App extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      initialRoute: FirebaseAuth.instance.currentUser == null
        ? '/login'
        : '/profile',
      routes: {
        '/login': (context) {
          return SignInScreen(
            actions: [
              AuthStateChangeAction<SignedIn>((context, state) {
                if (!state.user!.emailVerified) {
                  Navigator.pushNamed(context, '/verify-email');
                } else {
                  Navigator.pushReplacementNamed(context, '/profile');
                }
              }),
            ]
          );
        },
        '/profile': (context) => ProfileScreen(),
        '/verify-email': (context) => EmailVerificationScreen(
          actionCodeSettings: ActionCodeSettngs(...),
          actions: [
            EmailVerified(() {
              Navigator.pushReplacementNamed(context, '/profile');
            }),
            Cancel((context) {
              FlutterFireUIAuth.signOut(context: context);
              Navigator.pushReplacementNamed(context, '/');
            }),
          ],
        ),
      }
    )
  }
}
```

Once opened, it triggers a verification email to be sent and awaits for a dynamic link to be received by the app (on supported platforms).

## Using `EmailVerificatioController`

If you want to build a custom email verification screen, you could use `EmailVerificationController`:

```dart
class MyEmailVerificationScreen extends StatefulWidget {
  const MyEmailVerificationScreen({Key? key}) : super(key: key);

  @override
  State<MyEmailVerificationScreen> createState() =>
      _MyEmailVerificationScreenState();
}

class _MyEmailVerificationScreenState extends State<MyEmailVerificationScreen> {
  late final ctrl = EmailVerificationController(FirebaseAuth.instance)
    ..addListener(() {
      // trigger widget rebuild to reflect new state
      setState(() {});
    });

  @override
  void dispose() {
    ctrl.dispose();
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    switch (ctrl.state) {
      case EmailVerificationState.unresolved:
      case EmailVerificationState.unverified:
        return TextButton(
          onPressed: () {
            ctrl.sendVerificationEmail(
              Theme.of(context).platform,
              ActionCodeSettings(...),
            );
          },
          child: Text('Send verification email'),
        );
      case EmailVerificationState.dismissed:
        return Text("Ok, let's verify your email next time");
      case EmailVerificationState.pending:
      case EmailVerificationState.sending:
        return CircularProgressIndicator();
      case EmailVerificationState.sent:
        return Text('Check your email');
      case EmailVerificationState.verified:
        return Text('Email verified');
      case EmailVerificationState.failed:
        return Text('Failed to verify email');
    }
  }
}
```

## Other topics

- [EmaiAuthProvider](./auth/providers/email.md) - allows to register and sign in using email and password.
- [EmailLinkAuthProvider](./auth/providers/email-link.md) - allows to register and sign in using a link sent to email.
- [PhoneAuthProvider](./auth/providers/phone.md) - allows to register and sign in using a phone number
- [UniversalEmailSignInProvider](./auth/providers/universal-email-sign-in.md) - gets all connected auth providers for a given email.
- [OAuth](./oauth.md)
- [Localization](./auth/localization.md)
- [Theming](./auth/theming.md)
- [Navigation](./auth/navigation.md)