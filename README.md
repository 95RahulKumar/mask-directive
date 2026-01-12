# mask-directive

import { Directive, ElementRef, HostListener } from '@angular/core';

@Directive({
  selector: '[dateMask]',
  standalone: true,
})
export class DateMaskDirective {
  private lastKey = '';

  constructor(private el: ElementRef<HTMLInputElement>) {}

  @HostListener('keydown', ['$event'])
  onKeyDown(event: KeyboardEvent) {
    this.lastKey = event.key;
    
    if ((event.ctrlKey || event.metaKey) && event.key.toLowerCase() === 'v') {
      return;
    }

    const allowed = ['Backspace', 'Delete', 'ArrowLeft', 'ArrowRight', 'Tab', 'Control', 'Meta'];
    if (!/[0-9]/.test(event.key) && !allowed.includes(event.key)) {
      event.preventDefault();
    }
  }

  @HostListener('paste', ['$event'])
  onPaste(event: ClipboardEvent) {
    const pastedData = event.clipboardData?.getData('text') || '';
    if (!/^[0-9\-]*$/.test(pastedData)) {
    }
  }

  @HostListener('input')
  onInput() {
    const input = this.el.nativeElement;
    const value = input.value;
    let cursor = input.selectionStart ?? 0;
    const isDeleting = this.lastKey === 'Backspace' || this.lastKey === 'Delete';

    let segments = value.split('-');
    
    let day = (segments[0] || '').replace(/\D/g, '').slice(0, 2);
    let month = (segments[1] || '').replace(/\D/g, '').slice(0, 2);
    let year = (segments[2] || '').replace(/\D/g, '').slice(0, 4);

    if (day.length === 2) {
      if (+day > 31) day = '31';
      else if (+day === 0) day = '01';
    }
    if (month.length === 2) {
      if (+month > 12) month = '12';
      else if (+month === 0) month = '01';
    }

    let formatted = day;

    if (day.length === 2) {
      if (isDeleting && segments.length === 1) {
      } else {
        formatted += '-' + month;
      }
    } else if (segments.length > 1) {
      formatted += '-' + month;
    }

    if (month.length === 2) {
      if (isDeleting && segments.length <= 2) {
        // stay as 'DD-MM'
      } else {
        formatted += '-' + year;
      }
    } else if (segments.length > 2) {
      formatted += '-' + year;
    }

    const oldLength = value.length;
    input.value = formatted;

    if (!isDeleting && formatted.length > oldLength && (cursor === 2 || cursor === 5)) {
      cursor++;
    }

    input.setSelectionRange(cursor, cursor);
  }
}
