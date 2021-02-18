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

Schauen wir uns das folgende Beispiel an:

```
	public class Person
	{
		private decimal speed;
		private (decimal X, decimal Y) Position;
		private decimal volume;
		private string language;

		public Person(decimal speed, (decimal X, decimal Y) position, decimal volume, string language)
		{
			this.speed = speed;
			Position = position;
			this.volume = volume;
			this.language = language;
		}

		public void Speak(string text)
		{
      // Logic
		}

		public void Walk((decimal X, decimal Y) newPos)
		{
      // Logic
		}
	}

```

Wenn wir diese Klasse testen, könnte das ungefähr so aussehen:

```
	public class PersonTest
	{
		[Test]
		async public Task Walking_Works()
		{
			var sut = new Lochkreis(2, new Kreis(10, new Point(0, 0)), Loch.NewRundloch(5), new Winkel(0));

			var entityObject = sut.ToDxf();

			Assert.That(entityObject, Has.Count.EqualTo(2));
		}

		[Test]
		async public Task Speaking_Works()
		{
			var sut = new Lochkreis(2, new Kreis(10, new Point(0, 0)), Loch.NewRundloch(5), new Winkel(0));

			var entityObject = sut.ToDxf();

			Assert.That(entityObject, Has.Count.EqualTo(2));
		}
  }
```

