import { TestBed } from '@angular/core/testing';
import { HttpClientTestingModule, HttpTestingController } from '@angular/common/http/testing';
import { LoginService } from './login.service';
import { SessionService } from '../session/session.service';
import { CookieService } from 'ngx-cookie-service';
import { CacheService } from '../cache/cache.service';

describe('LoginService', () => {
  let service: LoginService;
  let httpMock: HttpTestingController;
  let sessionService: jasmine.SpyObj<SessionService>;
  let cookieService: jasmine.SpyObj<CookieService>;
  let cacheService: jasmine.SpyObj<CacheService>;

  beforeEach(() => {
    const sessionSpy = jasmine.createSpyObj('SessionService', ['endSession', 'setCurrentUser']);
    const cookieSpy = jasmine.createSpyObj('CookieService', ['delete']);
    const cacheSpy = jasmine.createSpyObj('CacheService', ['invalidateCache']);

    TestBed.configureTestingModule({
      imports: [HttpClientTestingModule],
      providers: [LoginService, 
        { provide: SessionService, useValue: sessionSpy },
        { provide: CookieService, useValue: cookieSpy },
        { provide: CacheService, useValue: cacheSpy },
      ]
    });

    service = TestBed.inject(LoginService);
    httpMock = TestBed.inject(HttpTestingController);
    sessionService = TestBed.inject(SessionService) as jasmine.SpyObj<SessionService>;
    cookieService = TestBed.inject(CookieService) as jasmine.SpyObj<CookieService>;
    cacheService = TestBed.inject(CacheService) as jasmine.SpyObj<CacheService>;
  });

  afterEach(() => {
    httpMock.verify();
  });

  it('should be created', () => {
    expect(service).toBeTruthy();
  });

  it('should login and set user in session', () => {
    const mockResponse = {
      success: true,
      userId: '123',
    };

    service.login().subscribe(response => {
      expect(response).toEqual(mockResponse.userId);
    });

    const req = httpMock.expectOne(`${environment.backendURI}/api/v2/auth/login`);
    expect(req.request.method).toBe('POST');
    req.flush(mockResponse);

    expect(sessionService.setCurrentUser).toHaveBeenCalledWith(mockResponse.userId);
  });

  it('should log out, invalidate cache and end session', () => {
    service.logout();
    
    expect(cacheService.invalidateCache).toHaveBeenCalled();
    expect(sessionService.endSession).toHaveBeenCalled();
  });
});






import { ComponentFixture, TestBed } from '@angular/core/testing';
import { CoursesService } from 'src/app/service/courses/courses.service';
import { UserService } from 'src/app/service/user/user.service';
import { BrowseCoursesSourcesComponent } from './browse-courses-sources.component';

describe('BrowseCoursesSourcesComponent', () => {
  let component: BrowseCoursesSourcesComponent;
  let fixture: ComponentFixture<BrowseCoursesSourcesComponent>;

  beforeEach(async () => {
    const coursesServiceSpy = jasmine.createSpyObj('CoursesService', ['getAllCourses']);
    const userServiceSpy = jasmine.createSpyObj('UserService', ['getUser']);

    await TestBed.configureTestingModule({
      declarations: [ BrowseCoursesSourcesComponent ],
      providers: [
        { provide: CoursesService, useValue: coursesServiceSpy },
        { provide: UserService, useValue: userServiceSpy },
      ]
    })
    .compileComponents();
  });

  beforeEach(() => {
    fixture = TestBed.createComponent(BrowseCoursesSourcesComponent);
    component = fixture.componentInstance;
    fixture.detectChanges();
  });

  it('should create', () => {
    expect(component).toBeTruthy();
  });

  it('should set points', () => {
    component.setPoints(100);
    expect(component.points).toEqual(100);
  });

  it('should set level', () => {
    component.setPoints(300);
    component.setLevel();
    expect(component.level).toEqual(3);
  });

  it('should set name', () => {
    component.setName('testName');
    expect(component.name).toEqual('testName');
  });

  it('should not set name if parameter is undefined', () => {
    component.setName(undefined);
    expect(component.name).toEqual("");
  });
});






import { TestBed } from '@angular/core/testing';
import { RouterTestingModule } from '@angular/router/testing';
import { ActivatedRouteSnapshot, RouterStateSnapshot } from '@angular/router';
import { AuthService } from '../service/auth/auth.service';
import { AuthGuard } from './auth.guard';
import { of } from 'rxjs';

describe('AuthGuard', () => {
  let guard: AuthGuard;
  let authService: jasmine.SpyObj<AuthService>;
  let routerSpy: jasmine.SpyObj<Router>;

  beforeEach(() => {
    const authSpy = jasmine.createSpyObj('AuthService', ['isAuthenticated']);
    const routerSpyObj = jasmine.createSpyObj('Router', ['navigate']);

    TestBed.configureTestingModule({
      imports: [RouterTestingModule],
      providers: [
        AuthGuard,
        { provide: AuthService, useValue: authSpy },
        { provide: Router, useValue: routerSpyObj }
      ]
    });

    guard = TestBed.inject(AuthGuard);
    authService = TestBed.inject(AuthService) as jasmine.SpyObj<AuthService>;
    routerSpy = TestBed.inject(Router) as jasmine.SpyObj<Router>;
  });

  it('should be created', () => {
    expect(guard).toBeTruthy();
  });

  it('should return true for an authenticated user', () => {
    authService.isAuthenticated.and.returnValue(of(true));
    const route = {} as ActivatedRouteSnapshot;
    const state = { url: '/protected' } as RouterStateSnapshot;

    guard.canActivate(route, state).subscribe(canActivate => {
      expect(canActivate).toBeTrue();
    });
  });

  it('should navigate to login for an unauthenticated user', () => {
    authService.isAuthenticated.and.returnValue(of(false));
    const route = {} as ActivatedRouteSnapshot;
    const state = { url: '/protected' } as RouterStateSnapshot;

    guard.canActivate(route, state).subscribe(canActivate => {
      expect(canActivate).toBeFalse();
      expect(routerSpy.navigate).toHaveBeenCalledWith(['/login']);
    });
  });
});

 

