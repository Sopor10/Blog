---
title: "When Testclasses get to big"
last_modified_at: 2021-02-18T10:40:00
categories:
  - Blog
tags:
  - Post Formats
---

Wie organisiert man die Tests, wenn es zu viele werden um sie in einer einzelnen Datei überblicken zu können?

[Hier](https://haacked.com/archive/2012/01/02/structuring-unit-tests.aspx/) wird pro Funktionalität eine genestete Klasse erstellt.
Das hilft auf jeden Fall weiter.
Die Lösung ist meiner Meinung nach allerdings nicht optimal, 
die Testklasse nicht kleiner wird.
Sie lässt sich aber durch IDE Unterstützung leichter bearbeiten (z.b. lassen sich die genesteten Klassen ausblenden).

Warum kommen wir überhaupt in die Situation, zu viele Tests in einer Klasse zu haben?

Ich glaube, dass ist ein Zeichen für eine Klasse mit zu großer Verantwortung.
Um die Testklassen klein zu halten, müssen wir auch die getesteten Klassen klein halten (oder weniger testen ;) ).

Schauen wir uns das folgende Beispiel an:

``` c#
public class Person
{
  private decimal speed;
  private (decimal X, decimal Y) position;
  private decimal volume;
  private string language;

  public Person(decimal speed, (decimal X, decimal Y) position, decimal volume, string language)
  {
    this.speed = speed;
    this.position = position;
    this.volume = volume;
    this.language = language;
  }

  public void Speak(string text)
  {
    // Logic
  }

  public void Walk((decimal X, decimal Y) ziel)
  {
    //speed is between 0 and 1
    var newX = (ziel.X - position.X) * speed + position.X;
    var newY = (ziel.Y - position.Y) * speed + position.Y;
    this.position = (X,Y);
  }
}

```

Wenn wir diese Klasse testen, könnte das ungefähr so aussehen:

``` c#
public class PersonTest
{
  [Test]
  async public Task Walking_Works()
  {
    // Test
  }
  //------- noch viele weitere Walking Tests
  [Test]
  async public Task Speaking_Works()
  {
    // Test
  }
  //------- noch viele weitere Speaking Tests
}
```

Die Logik der `Walk` Methode benötigt nicht alle Eigenschaften der Person, um ihre Entscheidungen zu treffen.

Das ist aus mehreren Gründen problematisch.
Zum einen führt es dazu, dass wir in den Tests Werte erstellen müssen, die nichts mit dem Ausgang des Tests zu tun haben.
So hat ie `language` der Person keinen Einfluss auf das Gehen.

Meiner Erfahrung nach führt das zu vielen Tests mit sinnlosen Werten, die man aber erstmal als solche erkennen muss.

Optimal wäre es, wenn nur die Informationen bekannt sind, die auch benötigt werden um die Aufgabe zu erfüllen.

## Ein Lösungsversuch

An dieser Stelle kann man nun die Logik des Gehens aus der Klasse Person herausziehen.
Das sieht dann wie folgt aus:

```c#
public class Walking
{
  private decimal speed;
  private (decimal X, decimal Y) position;
  public Walking(decimal speed, (decimal X, decimal Y) position)
  {
    this.position = position;
    this.speed = speed;
  }

  public (decimal X, decimal Y) Walk((decimal X, decimal Y) ziel)
  {
    //speed is between 0 and 1
    var newX = (ziel.X - position.X) * speed + position.X;
    var newY = (ziel.Y - position.Y) * speed + position.Y;
    return (X,Y);
  }
}

```

Diese neue Klasse enthält nur noch die benötigten Informationen zum Gehen.
Dadurch kann sie mit viel weniger Aufwand getestet werden.
Eine Änderung an den Testwerten hat einen direkten Einfluss auf das Testergebnis.


## Einbinden der Lösung

Wir wollen für die Nutzer der `Person` keinen Breaking-Change verursachen.
Daher dürfen wir die public Methoden nicht verändern:

``` c#
public void Walk((decimal X, decimal Y) ziel)
{
  this.position = new Walking(speed, position).Walk(ziel);
}
```

Auf diese Weise haben wir weiterhin die Walking Logik versteckt hinter der Person.
Die konkrete Implementierung ist aber nicht mehr an die Person selbst gebunden.

Nun kann man die Tests, die das Gehen betreffen noch in die passende Klasse verschieben.

Dies ist meiner Meinung nach eine deutlich bessere Variante, als die Logik in der Person zu lassen und die Tests zu den verschiedenen Aspekten in genestete Klassen zu stecken.