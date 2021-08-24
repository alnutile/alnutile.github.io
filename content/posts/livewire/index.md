---
title: "Odd Livewire Error(s)"
date: 2021-08-17T14:32:07-04:00
hero: /images/heros/hero-time.png
images: "images/"
menu:
  sidebar:
    name: "Odd Livewire Error(s)"
    weight: -275
tags: [laravel, livewire, note]
---

Just keeping track of some nuances to LiveWire

## initialData.fingerprint

The table with rows was not interating. The rows had two columns of for-each loops so
as always you need keys, but I think my mistake was not making the keys different
for either of them.

You can see the table below

```
@foreach( $imports as $import)
                <tr class="bg-emerald-200">

                    <td class="border px-8 py-4">{{ $import->id }}</td>
                    <td class="border px-8 py-4">{{ $import->name }}</td>
                    <td class="border px-8 py-4">{{ optional($import->source)->type }}</td>

                    <td class="border px-8 py-4">
                        <div>@if($import->file_path)
                            @livewire('download-import', ['import' => $import], key('download-' . $import->id))

                            @else
                            <div>no file</div>
                            @endif
                        </div>

                    </td>

                    <td class="border px-8 py-4">
                        @livewire("status-link", [
                        'import' => $import
                        ], key('link-' . $import->id))
                    </td>
                </tr>
                @endforeach
```

The keys

```
key('download-' . $import->id))
```

and

```
key('link-' . $import->id))
```

Did not work when I was using `key($import->id)`

The table and search all just stopped :(

Anyways hope this saves the future me an hour!
