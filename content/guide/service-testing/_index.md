---
title: Test tjeneste
description: Informasjon om testing av tjenester
tags: ["guide"]
weight: 100
---

## Preview-funksjonalitet
I UI-editoren er det bygget inn en Preview-funksjonalitet. Denne tillater tjenesteutvikler å se hvordan skjema vil se ut til slutt. 

**Preview** er tilgjengelig under **UX** i toppmenyen, eller via **Preview**-fanen dersom man allerede er i UI-editoren. 

I tillegg til å se hvordan skjema vil _se ut_, kan man også teste f.eks. API-kall, kalkuleringer/regler, valideringer og dynamikk
og se hvordan disse oppfører seg.

{{<figure src="ui-editor-preview.gif?width=1000" title="Preview">}}

## Kjør tjenesten i Runtime

Tjenesten kan testes med en testbruker i Runtime.

1. Gå til tjenestearbeidsflaten (velg tjenesten i toppmenyen eller gå inn til tjenesten via forsiden)
2. Under **Test**, velg {{<icon name="fa-play">}}-ikonet.
3. Velg testbruker fra listen
4. Velg eksisterende instans, eller start ny.

Tjenesten kan nå testes, all funksjonalitet som er beskrevet over i _Preview_ er også tilgjengelige for test i Runtime.

{{<figure src="runtime-test.gif?width=1000" title="Test tjeneste i Runtime">}}
