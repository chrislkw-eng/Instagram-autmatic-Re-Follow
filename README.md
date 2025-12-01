# Instagram-autmatic-Re-Follow
# Short automated Python / Playwright Instagram Re-Follow bot for gaining Follows


# REQUIEREMENT: pip install playwright


from playwright.sync_api import sync_playwright, TimeoutError
import time
import random


PROFILE_DIR = "insta_profile"   # Session wird hier gespeichert
START_URL = "https://www.instagram.com/"
wait_short = random.uniform(1, 2)

username = "YOUR_USERNAME"
password = "YOUR_PASSWORD"





def ensure_logged_in(page):
    """Prüft, ob wir eingeloggt sind."""
    page.goto(START_URL)

    # Warten bis Seite geladen ist
    try:
        page.wait_for_selector("nav", timeout=5000)
        return True
    except TimeoutError:
        return False


def do_initial_login(page, username, password):

    print("Führe Erst-Login durch...")
    login(page, username, password)
    print("Login erfolgreich!")


def login(page, username, password):

    page.goto("https://www.instagram.com/accounts/login/")

    # Cookies akzeptieren
    try:
        page.click("text=Only allow essential cookies", timeout=3000)
    except:
        pass
    try:
        page.click("text=Nur essenzielle Cookies erlauben", timeout=3000)
    except:
        pass

    # Warten bis Inputfelder sichtbar sind
    page.wait_for_selector("input[name='username']")

    # Login-Daten eingeben
    page.fill("input[name='username']", username)
    page.fill("input[name='password']", password)

    # Login-Button
    page.click("button[type='submit']")

    # NICHT networkidle → blockiert auf Instagram!
    page.wait_for_selector("nav", timeout=15000)  # Zeichen, dass eingeloggte UI sichtbar ist

    # Popups wegklicken
    for text in ["Nicht jetzt", "Not now", "Später"]:
        try:
            page.click(f"text={text}", timeout=2000)
        except:
            pass


def accept_cookies(page):
    try:
        page.click("text=Nur essenzielle Cookies erlauben", timeout=3000)
    except:
        pass
    try:
        page.click("text=Only allow essential cookies", timeout=3000)
    except:
        pass
    try:
        page.click("text=Allow essential cookies", timeout=3000)
    except:
        pass





def run():
    with sync_playwright() as p:
        browser = p.chromium.launch_persistent_context(PROFILE_DIR, headless=False)
        page = browser.new_page()

        if not ensure_logged_in(page):
            do_initial_login(page, username, password)

        target_profiles = [
            "https://www.instagram.com/cristiano/",
            "https://www.instagram.com/leomessi/",
            "https://www.instagram.com/follow._for._.follow._.back/",
            "https://www.instagram.com/instagram/",
            "https://www.instagram.com/selenagomez/",
        ]

        while True:
            for url in target_profiles:
                print(f"➡️ Öffne Profil: {url}")
                page.goto(url, wait_until="domcontentloaded")
                page.wait_for_selector("header", timeout=10000)
            
                button = page.locator("button:has(svg[aria-label*='Pfeil'])")
                if page.locator("button:has(svg[aria-label='„Pfeil nach unten“-Symbol'])").is_visible():

                        button.click()
                        print("1. Entfolgen Menü öffnen.")

                        page.click("text=Nicht mehr Folgen")
                        print("2. Entfolgen Klick.")

                        page.click("text=Folgen")
                        print("3. Refollow.")

                        #warte wegen anti bot
                        print("Warte 1-2 Sekunden wegen Anti-Bot Schutz.")
                        time.sleep(wait_short)
                elif page.locator("button:has-text('Folgen')").is_visible():
                        page.click("button:has-text('Folgen')")
                        
                        #warte wegen anti bot
                        print("Warte 1-2 Sekunden wegen Anti-Bot Schutz.")
                        time.sleep(wait_short)


if __name__ == "__main__":
    run()



