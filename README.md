# QuizLib
Libreria di gestione di Quizzettino

## Reference

Segue l'elenco dei principali elementi della libreria.

**Costruttore**
```
Quizzettino quiz = new Quizzettino();
```
**enum ButtonCode**

Rappresenta il codice del pulsante ed i parametri addizionali.
Pulsanti: Reset, Button1, Button2, Button3, Button4, Button5, Button6, OKButton, NOButton
Parametri: AutoReset, Sound, Config

**Color[] ButtonColor**

Restituisce il colore corrispondente al pulsante specificato nel parametro (tra 1 e 6), rispettivamente:
Color.Pink, Color.Blue, Color.Green, Color.Yellow, Color.Orange, Color.Red

**string PortName**

Nome della porta COM.

**int PortSpeed**

Velocità (baudrate) della porta COM. Default: 115200

**ButtonEvent**

Puntatore alla funzione di gestione degli eventi relativi ai pulsanti, che va definita come segue (il nome è ovviamente liberamente modificabile):
```
private void ButtonHandler(object? sender, Quizzettino.QuizzettinoButtonEventArgs e)
```
Tale funzione va associata agli eventi dei pulsanti tramite la proprietà "ButtonEvent":
```
quiz.ButtonEvent += ButtonHandler;
```
Il parametro "QuizzettinoButtonEventArgs" contiene nella proprietà "e.Button" il codice del pulsante premuto (vedi "ButtonCode") ed in "e.State" lo stato (true=premuto).

**StringEvent**

Puntatore alla funzione di gestione degli eventi dei quali Quizzettino fornisce un valore di tipo stringa:
```
private void StringHandler(object? sender, Quizzettino.QuizzettinoStringEventArgs e)
```
Tale funzione va associata agli eventi di tipo stringa tramite la proprietà "StringEvent":
```
quiz.StringEvent += StringHandler;
```
Il parametro "QuizzettinoStringEventArgs" contiene nella proprietà "e.Code" la tipologia di risposta ed in "e.Value" la stringa ricevuta. Attualmente questo evento viene richiamato solamente quando si richiedono i valori di configurazione, per i quali "e.Code" conterrà il carattere 'E' (per "EEPROM") ed "e.Value" i valori letti dai primi byte della EEPROM in formato esadecimale e separati da punto e virgola (es. "1;1;0").


## Codice di esempio

Segue qualche porzione di codice di esempio di uso della libreria, ma per ulteriori dettagli si consiglia di prendere visione del codice del progetto di esempio "QuizzettinoDemo".
```
  static Quizzettino quiz = new Quizzettino();
...
  // Apertura della porta
  try
  {
      quiz = new Quizzettino();
      // Codice per gli eventi
      quiz.ButtonEvent += ButtonHandler;
      quiz.StringEvent += StringHandler;
      // Imposto la porta
      quiz.PortName = (string)lstPorts.SelectedItem;
      quiz.PortSpeed = 115200;
      quiz.Open();
  }
  catch (Exception)
  {
      MessageBox.Show("Errore durante l'apertura della porta " + lstPorts.SelectedText);
      return;
  }
...
        private void ButtonHandler(object? sender, Quizzettino.QuizzettinoButtonEventArgs e)
        {
            if (sender == null) return;
            int c = (int)e.Button;
            if (c >= 1 && c <= 6)
            {
                // Il giocatore 'c' ha premuto il pulsante!
                lblGiocatore[c-1].BackColor = LabelColor(c);
                return;
            }
            bool value = e.State;
            switch (e.Button)
            {
                case Quizzettino.ButtonCode.Reset:
                    for (int g = 0; g <= 5; ++g)
                        lblGiocatore[g].BackColor = LabelColor(g);
                    break;
                case Quizzettino.ButtonCode.OKButton:
                    btnSI.BackColor = Color.Green;
                    Thread.Sleep(1000);
                    btnSI.BackColor = Color.DarkGreen;
                    break;
                case Quizzettino.ButtonCode.NOButton:
                    btnNO.BackColor = Color.Red;
                    Thread.Sleep(1000);
                    btnNO.BackColor = Color.DarkRed;
                    break;
                case Quizzettino.ButtonCode.AutoReset:
                    chkAutoReset.Checked = e.State;
                    break;
            }
        }
...
        private Color LabelColor(int c)
        {
            if (quiz.GetButton(c))
                return quiz.ButtonColor[c-1];
            else
                return Color.Gray;
        }

        private void ClickButton(Object? sender, EventArgs e)
        {
            if (sender == null) return;
            int c = int.Parse(((Button)sender).Text.ToString());
            quiz.SetButton(c, true);            
        }

```
