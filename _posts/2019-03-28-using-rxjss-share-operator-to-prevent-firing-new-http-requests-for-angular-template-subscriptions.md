---
layout: post
hidden: false
title: >-
  Using RXJS's shareReplay() Operator to Prevent Firing New HTTP Requests for
  Template Subscriptions
tags:
  - RxJs
  - shareReplay
  - Angular
  - Form Arrays
---
Just read this[ article](https://blog.angularindepth.com/rxjs-understanding-the-publish-and-share-operators-16ea2f446635) by [Nicholas Jamieson](https://blog.angularindepth.com/@cartant) which gave me a whole lot of new insights into how RxJs operator functions are written. I encourage you to have a look.

So I have this contact form component with multiple form arrays whose items are added dynamically from the template and during initialization.

```typescript
export class CreateContactComponent extends FormCanDeactivate implements OnInit {


  form: FormGroup;

  guids = DropdownGuids;
  categoryOptions$: Observable<Array<IServerDropdownOption>>;
  countryOptions$: Observable<Array<IServerDropdownOption>>;
  programOptions$: Observable<Array<IServerDropdownOption>>;
  bucketOptions$: Observable<Array<IServerDropdownOption>>;
  fileAsOptions$: Observable<IServerDropdownOption[]>;
  socialLabels$: Observable<IServerDropdownOption[]>;
  phoneLabels$: Observable<IServerDropdownOption[]>;
  emailLabels$: Observable<IServerDropdownOption[]>;
  websiteLabels$: Observable<IServerDropdownOption[]>;
  addressLabels$: Observable<IServerDropdownOption[]>;
  recurringEventLabels$: Observable<IServerDropdownOption[]>;

  @Input() formData: IMainContact;

  constructor(
    private fb: FormBuilder,
    private contactsService: ContactsService,
    private countryService: CountryService
  ) { }


  ngOnInit() {
    this.categoryOptions$ = this.contactsService.categoryList();
    this.countryOptions$ = this.countryService.fetchAll();
    this.programOptions$ = this.contactsService.programsList(this.guids.PROGRAMS_CONTACT);
    this.bucketOptions$ = this.contactsService.bucketList();
    this.socialLabels$ = this.contactsService.labelsList(this.guids.SOCIAL_LABELS);
    this.phoneLabels$ = this.contactsService.labelsList(this.guids.PHONE_LABELS);
    this.emailLabels$ = this.contactsService.labelsList(this.guids.EMAIL_LABELS);
    this.websiteLabels$ = this.contactsService.labelsList(this.guids.WEBSITE_LABELS);
    this.addressLabels$ = this.contactsService.labelsList(this.guids.ADDRESS_LABELS);
    this.recurringEventLabels$ = this.contactsService.labelsList(this.guids.RECURRING_EVENT_LABELS);

    this.initForm();
  }


  initForm() {
      const addresses = this.formData && this.formData.addresses
          ? this.formData.addresses.map(address => this.createAddress(address))
          : [this.createAddress()];

      const recurring_events = this.formData && this.formData.recurring_events
          ? this.formData.recurring_events.map(event => this.createRecurringEvent(event))
          : [this.createRecurringEvent()];

      const follow_ups = this.formData && this.formData.follow_ups
          ? this.formData.follow_ups.map(follow_up => this.createFollowUp(follow_up))
          : [this.createFollowUp()];

      const phones = this.formData && this.formData.phones
          ? this.formData.phones.map(phone => this.createLabeledPhoneInput(phone))
          : [this.createLabeledPhoneInput()];

      const buckets = this.formData && this.formData.buckets || [];

      const categories = this.formData && this.formData.categories || [];

    this.form = this.fb.group({
      first_name: [this.formData && this.formData.first_name],
      middle_name: [this.formData && this.formData.middle_name],
      last_name: [this.formData && this.formData.last_name],
        image: [this.formData && this.formData.image],
      dob: [this.formData && this.formData.dob],
      initials: [this.formData && this.formData.initials],
      spouse_name: [this.formData && this.formData.spouse_name],
      file_as: [this.formData && this.formData.file_as],
      company_name: [this.formData && this.formData.company_name],
      job_title: [this.formData && this.formData.job_title],
      phones: this.fb.array([...phones]),
      emails: this.fb.array([...this.labeledAddressInputsOrNew(this.formData && this.formData.emails)]),
      websites: this.fb.array([...this.labeledAddressInputsOrNew(this.formData && this.formData.websites)]),
      socials: this.fb.array([...this.labeledAddressInputsOrNew(this.formData && this.formData.socials)]),
      recurring_events: this.fb.array([...recurring_events]),
      addresses: this.fb.array([...addresses]),
      follow_ups: this.fb.array([...follow_ups]),
      notes: [this.formData && this.formData.notes],
      add_to_outlook: [this.formData && this.formData.add_to_outlook],
      categories: [[...categories]],
      buckets: [[...buckets]],
      programs: [this.formData && this.formData.programs],
    });
  }

createLabeledPhoneInput(data?: ILabeledPhoneInput): FormGroup {
      return this.fb.group({
          id: [data && data.id],
          type: [data && data.type],
          number: [data && data.number],
      });
  }

    labeledPhoneInputsOrNew(inputs: ILabeledPhoneInput[] = []): FormGroup[] {
        return inputs && inputs.length
            ? this.createLabeledPhoneInputs(inputs)
            : [this.createLabeledAddressInput()];
    }

  createLabeledPhoneInputs(inputs: ILabeledPhoneInput[]): FormGroup[] {
      return inputs.map(input => this.createLabeledPhoneInput(input));
  }

  createLabeledAddressInputs(inputs: ILabeledAddressInput[]): FormGroup[] {
      return inputs.map(input => this.createLabeledAddressInput(input));
  }

  labeledAddressInputsOrNew(inputs: ILabeledAddressInput[] = []): FormGroup[] {
      return inputs && inputs.length
          ? this.createLabeledAddressInputs(inputs)
          : [this.createLabeledAddressInput()];
  }
  createLabeledAddressInput(data?: ILabeledAddressInput) {
      return this.fb.group({
          id: [data && data.id],
          type: [data && data.type],
          address: [data && data.address],
      });
  }
  createAddress(data?: IContactAddress) {
      return this.fb.group({
          id: [data && data.id],
          type: [data && data.type],
          street: [data && data.street],
          unit: [data && data.unit],
          city: [data && data.city],
          state: [data && data.state],
          postal_code: [data && data.postal_code],
          country: [data && data.country],
      });
  }
  createRecurringEvent(data?: IRecurringEventContact) {
      return this.fb.group({
          id: [data && data.id],
          type: [data && data.type],
          date: [data && data.date],
          description: [data && data.description],
      })
  }
  createFollowUp(data?: IFollowUp) {
      return this.fb.group({
          id: [data && data.id],
          start_date: [data && data.start_date],
          end_date: [data && data.end_date],
          notes: [data && data.notes],
          method: [data && data.method],
      })
  }
}
```

Here's what the template for the phones form array looks like.

```html
<!-- PHONE -->
<div class="row">
    <div class="col-md-12">
        <h4 class="form-section__title">
            <igx-icon class="form-section__icon">phone</igx-icon> Phone
        </h4>

        <ng-container formArrayName="phones"
                      *ngFor="let phone of phones.controls; let i = index;">
            <div class="row">
                <div class="col-12">
                    <ng-container [formGroupName]="i">
                        <igx-input-group>
                            <igx-prefix class="prefix__label-select">
                                <app-dropdown formControlName="type"
                                              placeholder="Select label"
                                              (click)="$event.stopPropagation();"
                                              [options]="phoneLabels$ | async">

                                </app-dropdown>
                            </igx-prefix>
                            <input igxInput name="number" formControlName="number"
                                   type="text" placeholder="(123) 456-7890"/>

                            <igx-suffix class="button__remove" *ngIf="hasTwoOrMoreItems(phones)">
                                <button igxButton="icon" igxRipple type="button"
                                        (click)="removeFormArrayItem(i, phones); $event.stopPropagation();"
                                        igxRippleCentered="true">
                                    <igx-icon fontSet="material">close</igx-icon>
                                </button>
                            </igx-suffix>
                        </igx-input-group>
                    </ng-container>
                </div>
            </div>
        </ng-container>
    </div>
    <div class="col-md-12">
        <div class="x-small-space"></div>
        <button type="button" (click)="addFormArrayGroup(createLabeledPhoneInput(), phones)"
                igxButton="raised" igxButtonBackground="#f2f2f2" igxRipple class="btn-add">
            Add
        </button>
    </div>
</div>
```

On clicking the `Add` button, the `addFormArrayGroup(createLabeledPhoneInput(), phones)` method is called which creates and add a new item to the phones form array.

This however triggers the creation and subscription to a new phoneLabels$ observable for the phone type dropdown options. This is the behaviour I don't want given it triggers an http request to fetch the options.

```html
<app-dropdown formControlName="type"
              placeholder="Select label"
              (click)="$event.stopPropagation();"
              [options]="phoneLabels$ | async">

</app-dropdown>
```

Adding the `shareReplay` RxJs operator to the observable removes this behaviour. 

This operator is a specialization of `replay` that connects to a source observable and multicasts through a `ReplaySubject` constructed with the specified arguments.

For a more in-depth look into it's functioning, look [here](https://blog.angularindepth.com/rxjs-understanding-the-publish-and-share-operators-16ea2f446635).

So here's how my observables look after this update.

```typescript
this.categoryOptions$ = this.contactsService.categoryList().pipe(shareReplay());
this.countryOptions$ = this.countryService.fetchAll().pipe(shareReplay());
this.programOptions$ = this.contactsService.programsList(this.guids.PROGRAMS_CONTACT).pipe(shareReplay());
this.bucketOptions$ = this.contactsService.bucketList().pipe(shareReplay());
this.socialLabels$ = this.contactsService.labelsList(this.guids.SOCIAL_LABELS).pipe(shareReplay());
this.phoneLabels$ = this.contactsService.labelsList(this.guids.PHONE_LABELS).pipe(shareReplay());
this.emailLabels$ = this.contactsService.labelsList(this.guids.EMAIL_LABELS).pipe(shareReplay());
this.websiteLabels$ = this.contactsService.labelsList(this.guids.WEBSITE_LABELS).pipe(shareReplay());
this.addressLabels$ = this.contactsService.labelsList(this.guids.ADDRESS_LABELS).pipe(shareReplay());
this.recurringEventLabels$ = this.contactsService.labelsList(this.guids.RECURRING_EVENT_LABELS).pipe(shareReplay());
```
