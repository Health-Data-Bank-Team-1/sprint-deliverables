# Backend Framework Options

Option 1 - Laravel (PHP)

- Full stack PHP framework (provides everything for both back and frontend
    (Blade) in PHP)
- Supports MySQL
- Huge amount of documentation
- Innate support or package options for all of our core features (RBAC, 2FA, audit
    logging, CSV exporting, security, notifications, form handling)
- Meets the client’s language preferences
- Relatively simple deployment
- Overall, my personal recommendation for the project. Laravel provides all of the
    core features we need either out of the box or with the addition of a package or
    two (Breeze, Spatie), meaning development should proceed quickly and easily

Option 2 - Django (Python)

- Full stack python framework (With a caveat. Technically capable of frontend
framework, but from my research it’s generally recommended to pair with a
secondary, more modern frontend like htmx or VUE)
- Supports MySQL
- Also provides out of the box or package support for all of our core features
(RBAC, 2FA, audit logging, CSV exporting, security, notifications, form handling).
- Slightly more complex deployment
- A good second consideration for the project. Essentially provides the same
benefits as Laravel on the backend, but through python. Docked a few points for
needing a secondary frontend framework

Option 3 - Next.js (JavaScript/React)

- Full stack react framework
- More than likely will require a higher learning curve out of our team
- Very modern, highly in demand framework at the moment
- Provides many of the core features of our project out of the box, but will also
require a good bit more manual coding for features like the audit log and general
security
- I’ve mostly listed this option due to the popularity of the framework in the modern
workspace according to my research. If the client is comfortable with it it would
make for a great opportunity to learn a valuable set of skills - the tradeoff being it
would require more work and possibly be more out of the client’s comfort zone.

