import time
from collections import defaultdict

MAX_ATTEMPTS = 5          # allowed failures
LOCKOUT_TIME = 60         # seconds

failed_attempts = defaultdict(list)
locked_until = {}

def is_locked(identity):
    if identity in locked_until:
        if time.time() < locked_until[identity]:
            return True
        else:
            del locked_until[identity]
    return False

def record_failure(identity):
    now = time.time()
    failed_attempts[identity].append(now)

    # keep only recent attempts
    failed_attempts[identity] = [
        t for t in failed_attempts[identity]
        if now - t < LOCKOUT_TIME
    ]

    if len(failed_attempts[identity]) >= MAX_ATTEMPTS:
        locked_until[identity] = now + LOCKOUT_TIME
        failed_attempts[identity].clear()
        return True

    return False

def record_success(identity):
    failed_attempts[identity].clear()

# ----------------- Example usage -----------------

def login(identity, password):
    if is_locked(identity):
        print("🚫 Account temporarily locked.")
        return False

    if password != "secret123":   # replace with real check
        locked = record_failure(identity)
        print("❌ Login failed.")
        if locked:
            print("🔒 Too many attempts. Locked.")
        return False

    record_success(identity)
    print("✅ Login successful.")
    return True

# Demo
if __name__ == "__main__":
    while True:
        user = input("User: ")
        pwd = input("Password: ")
        login(user, pwd)
