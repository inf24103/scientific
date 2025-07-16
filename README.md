1) Riemann-Integration:
def berechne(self, f, a, b, n, method='trapez'): <-- Methode aufrufen mit f als funktion, Intervall [a, b], n Unterteilungen und method als Methode.

Hier wird die Intervallaufteilung bestimmt mit:
x = np.linspace(a, b, n+1)  Zahlenfolge a bis b mir n+1 Anzahl der Punkte
dx = (b - a) / n    Breite des Intervalls
y = f(x)          Funktionswerte als f(x)

Das überprüft, ob irgendeine FUnktion 0 ist und gibt Fehler, falls es der Fall ist:
if np.any(np.isnan(y)) or np.any(np.isinf(y)):
    raise ValueError("Funktionswerte enthalten NaN oder Inf (Definitionslücken?)")

Warnung falls eine Funktion Null-Durchgänge hat:
if np.any(y == 0):
    print("Achtung: Funktionswerte enthalten Null-Durchgänge.")
    pass

Hier werden methoden unter/ober/trapez bestimmt für Untersumme, Obersumme, Trapezregel und die entsprechende Formel verwendet.
if method == 'unter':
    result = sum(min(y[i], y[i+1]) for i in range(n)) * dx <-- Minimale Summe (Rechtecke unterhalb der Funktion)
elif method == 'ober':
    result = sum(max(y[i], y[i+1]) for i in range(n)) * dx <-- Maximale Summe (Rechtecke oberhalb der Funktion)
elif method == 'trapez':
    result = dx * (np.sum(y) - 0.5 * (y[0] + y[-1])) <-- Summe an der Funktion angepasst.


else:
    raise ValueError("Unbekannter Integrationstyp. Wähle 'unter', 'ober' oder 'trapez'.") <--- Falls die Methode unbekannt ist

class TestTeil1Riemann(unittest.TestCase):
Die Funktionen in der Klassen rufen einfach Funktionen auf mit den Eingaben, die dann in der Riemann-Integration Klasse verwendet werden.
Das hier ist eine e^x Funktion mit I = [0, 10] und trapez Methode.
def test_exponential_trapez(self):
        f = np.exp
        F = lambda x: np.exp(x)
        a, b = 0, 10
        exact = F(b) - F(a)
        trapez = self.intg.berechne(f, a, b, 1000, method='trapez')
        self.assertAlmostEqual(trapez, exact, delta=0.2) <-- Das delta gilt als Abweichung zwischen trapez und exakt


2) Monte-Carlo-Integration
def berechne(self, f, a, b, n=10000, explorative_samples=1000): <--- Aufruf der Funktion mit f als Funktion, Intervall [a, b], n Anzahl zufälliger Punkte, explorative_sample Wird genutzt, um y_min und y_max abzuschätzen

Erstellung gleichmäßiger Stichproben im Intervall [a,b]
x_probe = np.linspace(a, b, explorative_samples)
y_probe = f(x_probe)
ymin = np.min(y_probe)
ymax = np.max(y_probe)
ymin und ymax ist für Höhe der Zufallspunkte zu definieren.

x_rand = np.random.uniform(a, b, n) <-- Zufällige x-Werte werden generiert

Fläche über der x_Achse wird berechnet:
if ymax > 0:
    y_rand_pos = np.random.uniform(0, ymax, n)
    f_vals_pos = f(x_rand)
    unter_kurve_pos = y_rand_pos <= f_vals_pos
    anteil_pos = np.count_nonzero(unter_kurve_pos) / n
    flaeche_pos = (b - a) * ymax

Flächse unter der x_Achse wird berechnet:
if ymin < 0:
    y_rand_neg = np.random.uniform(ymin, 0, n)
    f_vals_neg = f(x_rand)
    unter_kurve_neg = y_rand_neg >= f_vals_neg
    anteil_neg = np.count_nonzero(unter_kurve_neg) / n
    flaeche_neg = (b - a) * abs(ymin)
    
return anteil_pos * flaeche_pos - anteil_neg * flaeche_neg <-- Ergebnis berechnen

3) Visualisierung
Funktionen werden definiert:

f_sin = np.sin
f_parabel = lambda x: x**2 - 4*x + 2
f_exp = np.exp

Intervalle der Funktionen:
interval_sin = (0, 2 * np.pi)
interval_parabel = (-3, 3)
interval_exp_riemann = (0, 10)
interval_exp_monte = (0, 3)

Riemann visualisieren
def plot_riemann(f, a, b, method='trapez', n=100, title=''):

Monte-Carlo visualisieren
def plot_monte_carlo(f, a, b, n=10000, title=''):


4) Test auf numerische Integration
Der Code testet Funktion f auf Konvergenz auf Intervall [a,b]
def _konvergenz_test(self, f, a, b, name):
    n_values = [2, 4, 8, 16, 32, 64, 128, 256]  <--- sind verschiedene Anzahl an Unterteilungen, mit denen das Integral approximiert wird.
    exact_value, _ = quad(f, a, b)  <-- berechnet den genauen Wert des Integrals mit SciPy

    print(f"\n--- Konvergenztest für {name} auf [{a}, {b}] ---")
    print(f"Exakter Wert (scipy.quad): {exact_value:.8f}")

    for n in n_values:        <--- Jedes Integral wird n-Mal numerisch berechnet und der Fehler zum exakten Wert ausgegeben.
        approx = self.intg.berechne(f, a, b, n, method='trapez')
        fehler = abs(approx - exact_value)
        print(f"n = {n:<3} → Trapezregel = {approx:.8f} | Fehler = {fehler:.2e}") <-- Ausgabe, wie die Annäherung mit zunehmendem n genauer wird.

5) Grenzen numerischer Integration
class TestTeil5OszillierendeFunktion(unittest.TestCase):
    def setUp(self):
        self.intg = RiemannIntegration()  <-- Holt sich Objekt aus der RiemannIntegration Klasse

Hoch oszillierende Funktionen werden hier getestet:
def test_hochoszillierend(self):
    f = lambda x: np.sin((x + 1/32) * 16 * np.pi)  <--- Die Funktion aus der Aufgabe wird hier getestet
    a, b = 0, 1    <--- Intervall [0,1]
    exact, _ = quad(f, a, b)  <-- Berechnung von genauen Wert mithilfe von quad

    print("\nVergleich für f(x) = sin((x + 1/32) * 16π) auf [0, 1]:")
    print(f"Echter Wert mit scipy.quad: {exact:.6f}")
    print(f"{'n':>4} {'Trapez':>12} {'Untersumme':>12} {'Obersumme':>12}")

    for n in [2, 4, 8, 16]:        <--- Hier werden 3 Methoden überprüft: Trapezregel, Untersumme, Obersumme
        t = self.intg.berechne(f, a, b, n, method='trapez')     Trapezregel ist oft genauer
        u = self.intg.berechne(f, a, b, n, method='unter')
        o = self.intg.berechne(f, a, b, n, method='ober')
        print(f"{n:4d} {t:12.6f} {u:12.6f} {o:12.6f}")
Bei zu kleinem n werden Oszillationen nicht erfasst → schlechte Annäherung
